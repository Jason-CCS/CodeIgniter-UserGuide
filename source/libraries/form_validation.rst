###############
表單驗證
###############

CodeIgniter 提供一個全面的表單驗證功能與資料預處理類別，它可以幫忙簡化你撰寫程式的數量

.. contents:: Page Contents

********
Overview
********

在解釋表單驗證功能之前，我們先描述一個理想的表單驗證場景：

#. 一個表單已經完成
#. 你填寫好這個表單並且提交它
#. 如果你有什麼地方填寫錯誤或者是漏填必要的資料時，這個表單會展示你原本的資料並且顯示訊息說明填寫錯誤的原因
#. 這整個過程一直持續到你提交一個正確的表單資料

在伺服器接收方，資料必須：

#. 檢查必填的資料
#. 驗證該資料的型別與確認該資料的內容格式要求；舉例來說，註冊的使用者名稱只能包含被允許的字元，它有最短長度，以及最常長度的要求。
   還有， username 不能與任何已存在的其他使用者名稱相同，也不能使用系統地保留字，等等其他類似要求。
#. 為資料的安全性消毒。
#. 對資料做格式化（比方說，資料是否需要修剪多餘字元？對 HTML 做編碼？ 等其他類似處理。）
#. 插入資料庫前對資料做預處理

雖然以上的過程並不會太複雜，但是實際上對這些資料做處理卻需要大量的程式碼，顯示錯誤訊息的功能，多個控制結構被置放於 HTML 表單中。
表單驗證功能，雖然表單容易製作，但是實作上是麻煩且繁複的的過程。

************************
表單驗證教學
************************

接下來是一個容易上手的表單驗證教學

要實作表單驗證你將需要以下這三件事:

#. :doc:`View <../general/views>` 一個擁有表單的檔。
#. 一個 View file 呈現成功訊息，當表單有順利的送出時。
#. :doc:`controller <../general/controllers>` 一個控制器來接收與處理由表單送出的資料。

讓我們使用會員註冊表單作為範例，實作這三個檔案。

The Form
========

使用文字編輯器，建立一個表單叫做 myform.php。 其中，放入以下這段程式碼並儲存到 application/views/ folder::

	<html>
	<head>
	<title>My Form</title>
	</head>
	<body>

	<?php echo validation_errors(); ?>

	<?php echo form_open('form'); ?>

	<h5>Username</h5>
	<input type="text" name="username" value="" size="50" />

	<h5>Password</h5>
	<input type="text" name="password" value="" size="50" />

	<h5>Password Confirm</h5>
	<input type="text" name="passconf" value="" size="50" />

	<h5>Email Address</h5>
	<input type="text" name="email" value="" size="50" />

	<div><input type="submit" value="Submit" /></div>

	</form>

	</body>
	</html>

The Success Page
================

使用文字編輯器，建立一個頁面叫做 formsuccess.php。 其中，放入以下這段程式碼並儲存到 application/views/ folder::

	<html>
	<head>
	<title>My Form</title>
	</head>
	<body>

	<h3>Your form was successfully submitted!</h3>

	<p><?php echo anchor('form', 'Try it again!'); ?></p>

	</body>
	</html>

The Controller
==============

使用文字編輯器，建立一個控制器叫做 Form.php。 其中，放入以下這段程式碼並儲存到 application/controllers/ folder::

	<?php

	class Form extends CI_Controller {

		public function index()
		{
			$this->load->helper(array('form', 'url'));

			$this->load->library('form_validation');

			if ($this->form_validation->run() == FALSE)
			{
				$this->load->view('myform');
			}
			else
			{
				$this->load->view('formsuccess');
			}
		}
	}

Try it!
=======

試看看你的表單，訪問你的網站，使用如下相似的 url 來訪問你的網站::

	example.com/index.php/form/

當你送出表單時，你將只會看到表單被重新載入，這是因為你還沒有設置任何的表單驗證規則。

**因為你還沒有設置任何的表單驗證規則，預設它會回傳 FALSE。當你送出的表單通過所有的驗證，``The run()``方法將會回傳 TRUE。

解  說
===========

在上方的範例，你會注意到幾件事:

這個表單是標準的網頁表單除了:

#. 它使用表單輔助函式來建立表單的起始標籤。在技術上您是不需要如此做的，你可以使用標準的 HTML 語法來建立，
   然而，使用輔助函式的好處在於它會自動依據您在 config file 設定的 URL 來配置你的 action URL，這將使此
   網站更便於轉移在不同的主機之間，即便您的 URL 有所改變。
#. 在表單的最上面你會注意到一個函式呼叫:
   ::

	<?php echo validation_errors(); ?>

   當驗證規則有誤時，這個函式將會回傳任何的錯誤資訊，如果沒有錯誤則回傳空字串。

控制器有個方法叫 ``index()``。 這個方法初始化驗證類與載入表單輔助函式與 URL 輔助函式，
此控制器也執行驗證過程，由驗證過程的正確與否來展示表單或是成功頁面。

.. _setting-validation-rules:

設定驗證規則
========================

CodeIgniter 可以讓你對單一個欄位依需求的進行多種驗證，並同時給予你對欄位資料進行預準備或前置準備，
為了設置驗證規則你將會使用 ``set_rules()``方法::

	$this->form_validation->set_rules();

上面的方法會設定三個參數數入:

#. 欄位名 - 你為表單欄位所取的名字。
#. 為這個欄位取一個人看得懂的名字，這個名字將會在錯誤出現時插入到錯誤訊息中。舉例來說，欄位名如果是 user，你可能提供更易於人解讀的名字 “Username。
#. 對此欄位的驗證規則。
#. （可選） 在此欄位設定自訂的錯誤訊息，如果沒有設定將會使用預設值。

.. note:: 如果你想要欄位名被儲存在語言檔案裡，請參考
	:ref:`translating-field-names`.

這裡有個範例。在你的控制器中 (Form.php)，增加以下這些程式碼在驗證初始化方法的下面::

	$this->form_validation->set_rules('username', 'Username', 'required');
	$this->form_validation->set_rules('password', 'Password', 'required');
	$this->form_validation->set_rules('passconf', 'Password Confirmation', 'required');
	$this->form_validation->set_rules('email', 'Email', 'required');

你的控制器現在應該看起來如下::

	<?php

	class Form extends CI_Controller {

		public function index()
		{
			$this->load->helper(array('form', 'url'));

			$this->load->library('form_validation');

			$this->form_validation->set_rules('username', 'Username', 'required');
			$this->form_validation->set_rules('password', 'Password', 'required',
				array('required' => 'You must provide a %s.')
			);
			$this->form_validation->set_rules('passconf', 'Password Confirmation', 'required');
			$this->form_validation->set_rules('email', 'Email', 'required');

			if ($this->form_validation->run() == FALSE)
			{
				$this->load->view('myform');
			}
			else
			{
				$this->load->view('formsuccess');
			}
		}
	}

現在送出空白表單，沒有填入任何資料，你將會看到錯誤訊息。如果你送的表單有資料，那麼你會看到成功的頁面。

.. note:: 發生驗證錯誤時，表單欄位還未被正式重新填上舊資料，以下將繼續。

使用陣列來設定驗證規則
============================

再繼續之前，我們可以使用陣列的方式來設定驗證規則，使用這個方法，你必須設定好你的 'field'。

	$config = array(
		array(
			'field' => 'username',
			'label' => 'Username',
			'rules' => 'required'
		),
		array(
			'field' => 'password',
			'label' => 'Password',
			'rules' => 'required',
			'errors' => array(
				'required' => 'You must provide a %s.',
			),
		),
		array(
			'field' => 'passconf',
			'label' => 'Password Confirmation',
			'rules' => 'required'
		),
		array(
			'field' => 'email',
			'label' => 'Email',
			'rules' => 'required'
		)
	);

	$this->form_validation->set_rules($config);

疊加驗證規則
===========

CodeIgniter 可以讓你同時有多個驗證規則。在第三個參數設定組合的驗證規則，如下::

	$this->form_validation->set_rules(
		'username', 'Username',
		'required|min_length[5]|max_length[12]|is_unique[users.username]',
		array(
			'required'	=> 'You have not provided %s.',
			'is_unique'	=> 'This %s already exists.'
		)
	);
	$this->form_validation->set_rules('password', 'Password', 'required');
	$this->form_validation->set_rules('passconf', 'Password Confirmation', 'required|matches[password]');
	$this->form_validation->set_rules('email', 'Email', 'required|valid_email|is_unique[users.email]');

以上的程式碼設定以下的規則:

#. 用戶名欄位最短為 5 個字元，最常為 12 個字元。
#. 密碼欄位必須符合密碼驗證規則。
#. 信箱欄位必須包含有效的信箱地址表達。

試看看！在沒有填入正確的表單送出時，你將會看到新的錯誤訊息。你可以在 validation reference 查到許多的驗證規則。

.. note:: 你可以傳送一個陣列的驗證規則到``set_rules()``來取代使用字串的方式設定，例如::

	$this->form_validation->set_rules('username', 'Username', array('required', 'min_length[5]'));

資料預處理
=============

除了以上我們所設定的驗證方法，你可以對資料進行預處理。舉裡來說，你可以設定一些規則如下::

	$this->form_validation->set_rules('username', 'Username', 'trim|required|min_length[5]|max_length[12]');
	$this->form_validation->set_rules('password', 'Password', 'trim|required|min_length[8]');
	$this->form_validation->set_rules('passconf', 'Password Confirmation', 'trim|required|matches[password]');
	$this->form_validation->set_rules('email', 'Email', 'trim|required|valid_email');

在上面的例子，我們使用 "trimming" 來清除資料前後多餘的空百，並且檢查資料的長度，確認第二次輸入的密碼是否符合第一次輸入的密碼。

**任何原生的 PHP 函式如果只接受一個參數，那麼這些函數或方法都可以被拿來使用為驗證規則，如``htmlspecialchars()``, ``trim()``, 等等其他。**

.. note:: 你可能在資料驗證後才使用預處理，所以當有錯誤時，原始的資料可以被重新展現。

重新填入資料
======================

到目前為止，我們只有處理錯誤而已。當錯誤發生時，我們必須重新填入資料到表單欄位中。CodeIgniter 提供許多個輔助函式來完成這件事，
最常被使用到的是::

	set_value('field name')

打開你的 myform.php 視圖，然後使用 :php:func:`set_value()` 函式來更新每個欄位的**value**:

**不要忘記要包含欄位名稱當使用 :php:func:`set_value()` 的時候**

::

	<html>
	<head>
	<title>My Form</title>
	</head>
	<body>

	<?php echo validation_errors(); ?>

	<?php echo form_open('form'); ?>

	<h5>Username</h5>
	<input type="text" name="username" value="<?php echo set_value('username'); ?>" size="50" />

	<h5>Password</h5>
	<input type="text" name="password" value="<?php echo set_value('password'); ?>" size="50" />

	<h5>Password Confirm</h5>
	<input type="text" name="passconf" value="<?php echo set_value('passconf'); ?>" size="50" />

	<h5>Email Address</h5>
	<input type="text" name="email" value="<?php echo set_value('email'); ?>" size="50" />

	<div><input type="submit" value="Submit" /></div>

	</form>

	</body>
	</html>

現在，重新載入你的頁面，並且送出一個表單資料使其產生錯誤，你會發現你的表單欄位資料已竟被重新填上了。

.. note:: The :ref:`class-reference` section below
	contains methods that permit you to re-populate <select> menus,
	radio buttons, and checkboxes.

.. important:: 如果你是用陣列的方式作為表單欄位的名字，那麼你必須提供相同的陣列型態，如下範例::

	<input type="text" name="colors[]" value="<?php echo set_value('colors[]'); ?>" size="50" />

需要更多的資訊，請參考 :ref:`using-arrays-as-field-names`。

回呼: 你自己的驗證方法
======================================

驗證類別可以接受回呼函式來當作客製的驗證規則。舉例來說，如果你需要查詢資料庫來確定使用者輸入的是唯一的帳戶名，
你可以撰寫一個回呼方法，來完成這件事，來看看以下範例:

在控制器當中，改變 username 的規則為 'callback_username_check'::

	$this->form_validation->set_rules('username', 'Username', 'callback_username_check');

然後增加新的函式叫做 ``username_check()`` 在控制器當中，以下是你目前控制器應該的樣子::

	<?php

	class Form extends CI_Controller {

		public function index()
		{
			$this->load->helper(array('form', 'url'));

			$this->load->library('form_validation');

			$this->form_validation->set_rules('username', 'Username', 'callback_username_check');
			$this->form_validation->set_rules('password', 'Password', 'required');
			$this->form_validation->set_rules('passconf', 'Password Confirmation', 'required');
			$this->form_validation->set_rules('email', 'Email', 'required|is_unique[users.email]');

			if ($this->form_validation->run() == FALSE)
			{
				$this->load->view('myform');
			}
			else
			{
				$this->load->view('formsuccess');
			}
		}

		public function username_check($str)
		{
			if ($str == 'test')
			{
				$this->form_validation->set_message('username_check', 'The {field} field can not be the word "test"');
				return FALSE;
			}
			else
			{
				return TRUE;
			}
		}

	}

重新載入你的表單，然後送出 test 當作帳戶名，你會看到你送出的資料被送到回呼函式來進行處理。

為了調用回呼函式，只要在規則中放入你的方法名，並增加 "callback\_" 作為 **prefix**。如果你需要接受額外的參數到你的回呼函式，
只要在方法名的後面加上由中括號包起來的參數名，如此該參數就會作為的二個參數來調用你所呼叫的方法，如同 "callback_foo**[bar]**" 這樣子。

.. note:: 你也可以在你的回呼函式回傳值。如果你的回呼函式回傳的不是布林值，那麼該值將成為你該欄位的新值。

使用任何可呼叫的方法作為驗證規則
================================

如果回呼函式還不足以滿足需求（比方說，該函式被限制宣告在某一控制器當中），不必覺得失望，還有一個方法可以來做客製化的驗證規則。
limited to your controller), don't get disappointed, there's one more way
to create custom rules: anything that ``is_callable()`` would return TRUE for.

參考下面這個範例::

	$this->form_validation->set_rules(
		'username', 'Username',
		array(
			'required',
			array($this->users_model, 'valid_username')
		)
	);

上面的程式碼使用 ``Users_model`` 物件中的 ``valid_username()`` 方法來當作驗證規則。

這當然只是一個範例，回呼函式並不只限制在 models 中，你可以使用物件或方法的第一個參數來接受表單值並作為驗證規則，
或者在 PHP 5.3+，你也可以使用匿名方法，如下::

	$this->form_validation->set_rules(
		'username', 'Username',
		array(
			'required',
			function($value)
			{
				// Check $value
			}
		)
	);

當然，一個可呼叫的驗證規則其本身並不是一個字串，也不是一個規則的名字，則將會在設置錯誤訊息時發生麻煩。
處理這個麻煩的方法是，你可以使用一個陣列，第一個參數放置驗證規則的名字，第二個參數則放置驗證規則的方法::

	$this->form_validation->set_rules(
		'username', 'Username',
		array(
			'required',
			array('username_callable', array($this->users_model, 'valid_username'))
		)
	);

Anonymous function (PHP 5.3+) version::

	$this->form_validation->set_rules(
		'username', 'Username',
		array(
			'required',
			array(
				'username_callable',
				function($str)
				{
					// Check validity of $str and return TRUE or FALSE
				}
			)
		)
	);

.. _setting-error-messages:

設置錯誤訊息
===========

所有的原生錯誤訊息被放置在: **system/language/english/form_validation_lang.php**

要設置您個人的全域錯誤訊息，你可以使用 **application/language/english/form_validation_lang.php** 來擴展/覆載新的語言規則，
（閱讀更多相關文件 :doc:`Language Class <language>`），或是使用以下的方法::

	$this->form_validation->set_message('rule', 'Error Message');

如果你需要對一個特定欄位來設置一個自訂的錯誤訊息，使用 set_rules() method::

	$this->form_validation->set_rules('field_name', 'Field Label', 'rule1|rule2|rule3',
		array('rule2' => 'Error Message on rule2 for this field_name')
	);

其中 rule 代表某一個特定的規則，而錯誤訊息是一串文字，其將在發生驗證錯誤的時候顯示。

如果你喜歡在錯誤訊息中包含對人類友好的欄位名（也就是第二個參數），或是一些非必要規則的規則名與其設定值，
你可以加上 **{field}** 和 **{param}** 至你的錯誤訊息中，如下程式碼::

	$this->form_validation->set_message('min_length', '{field} must have at least {param} characters.');

對人類友好的欄位名與其最短長度 5 的規則下，錯誤將會顯示："Username must have at least 5 characters."

.. note:: The old `sprintf()` method of using **%s** in your error messages
	will still work, however it will override the tags above. You should
	use one or the other.

以上對回呼函式形式的規則進行錯誤訊息設置，並不需要 "callback\_" 前綴::

	$this->form_validation->set_message('username_check')

.. _translating-field-names:

欄位名的翻譯
=======================

如果你想要將對人類有好的翻譯名儲存在語言檔案中，使得相同欄位名的欄位可以使用相同的翻譯名，以下說明如何設定：

第一，使用 **lang:** 前綴你的人類友好參數，如下範例::

	 $this->form_validation->set_rules('first_name', 'lang:first_name', 'required');

然後，在語言檔案中儲存沒有前綴的翻譯名於陣列中::

	$lang['first_name'] = 'First Name';

.. note:: 如果你儲存的語言檔沒有被 CI 自動載入，你可以在控制器手動載入::

	$this->lang->load('file_name');

參考 :doc:`Language Class <language>` 頁面來獲取更多語言檔的說明。

.. _changing-delimiters:

改變錯誤定義符
=============================

預設中，Form Validation class 會增加一個段落標籤 (<p>) 於每一個錯誤訊息的外圍。
你可以全域的、個別的、或於設定檔中修改這些定義符。

#. **Changing delimiters Globally**
   要全域的修改錯誤定義符，於你的控制器載入 Form Validation class 後，增加定義符如下::

      $this->form_validation->set_error_delimiters('<div class="error">', '</div>');

   在這個範例中，我們改用 <div> 標籤來作為定義符。

#. **Changing delimiters Individually**
   此教學的兩個錯誤函式都可以個別設定錯誤定義符如下::

      <?php echo form_error('field name', '<div class="error">', '</div>'); ?>

   Or::

      <?php echo validation_errors('<div class="error">', '</div>'); ?>

#. **Set delimiters in a config file**
   你可以設定自己的錯誤定義符於 application/config/form_validation.php 如下::

      $config['error_prefix'] = '<div class="error_prefix">';
      $config['error_suffix'] = '</div>';

依欄位地顯示錯誤
===========================

如果你偏好將錯誤訊息顯示在欄位旁邊，你可以使用 :php:func:`form_error()` 函式。

試看看，將你的表單改變如下::

	<h5>Username</h5>
	<?php echo form_error('username'); ?>
	<input type="text" name="username" value="<?php echo set_value('username'); ?>" size="50" />

	<h5>Password</h5>
	<?php echo form_error('password'); ?>
	<input type="text" name="password" value="<?php echo set_value('password'); ?>" size="50" />

	<h5>Password Confirm</h5>
	<?php echo form_error('passconf'); ?>
	<input type="text" name="passconf" value="<?php echo set_value('passconf'); ?>" size="50" />

	<h5>Email Address</h5>
	<?php echo form_error('email'); ?>
	<input type="text" name="email" value="<?php echo set_value('email'); ?>" size="50" />

只有在有錯誤的時候，該錯誤訊息才會出現，否則不會。

.. important:: 如果你有使用陣列於欄位名中，你也必須提供相同的格式在函式中。例如::

	<?php echo form_error('options[size]'); ?>
	<input type="text" name="options[size]" value="<?php echo set_value("options[size]"); ?>" size="50" />

需要更多的資訊請參考 :ref:`using-arrays-as-field-names`。

驗證陣列
=======================================

有時候你會想驗證陣列，而非從 ``$_POST`` 資料而來。

在這個案例中，你可以驗證指定的陣列。

	$data = array(
		'username' => 'johndoe',
		'password' => 'mypassword',
		'passconf' => 'mypassword'
	);

	$this->form_validation->set_data($data);

創建驗證規則，執行驗證，這跟驗證 ``$_POST`` 資料是一樣的。

.. important:: 你必須在設定驗證規則前呼叫 ``set_data()`` 方法。

.. important:: 如果你想要驗證一個以上的資料陣列，那麼你先呼叫 ``reset_validation()`` 方法，再設定驗證規則與執行驗證。

需要更多的資訊請參考 :ref:`class-reference`。

.. _saving-groups:

************************************************
在配置檔中儲存驗證規則
************************************************

Form Validation class 的一個功能是允許可以儲存你所有的驗證規則至一個配置檔中，
你可以將這些驗證規則進行分組。這些驗證組別可以被自動的載入控制器或是被手動的載入。

如何儲存你的驗證規則
======================

如要儲存你自訂的驗證規則，只要建立一個檔案 form_validation.php，將此檔案儲存於 application/config 資料夾。
於該檔案中放入命名為 $config 的陣列，如下所示的驗證規則雛形::

	$config = array(
		array(
			'field' => 'username',
			'label' => 'Username',
			'rules' => 'required'
		),
		array(
			'field' => 'password',
			'label' => 'Password',
			'rules' => 'required'
		),
		array(
			'field' => 'passconf',
			'label' => 'Password Confirmation',
			'rules' => 'required'
		),
		array(
			'field' => 'email',
			'label' => 'Email',
			'rules' => 'required'
		)
	);

你的驗證規則將會被自動載入與執行 ``run()`` 方法的時候被使用。
Your validation rule file will be loaded automatically and used when you
call the ``run()`` method.

Please note that you MUST name your ``$config`` array.

創建多組驗證規則
======================

為了實現多組驗證規則，你必須於該驗證規則陣列中加上”子陣列“，如下範例展示兩組驗證規則。
我們將該兩組驗證規則命名為"signup" and "email"，你可以任意的取其他的名字::

	$config = array(
		'signup' => array(
			array(
				'field' => 'username',
				'label' => 'Username',
				'rules' => 'required'
			),
			array(
				'field' => 'password',
				'label' => 'Password',
				'rules' => 'required'
			),
			array(
				'field' => 'passconf',
				'label' => 'Password Confirmation',
				'rules' => 'required'
			),
			array(
				'field' => 'email',
				'label' => 'Email',
				'rules' => 'required'
			)
		),
		'email' => array(
			array(
				'field' => 'emailaddress',
				'label' => 'EmailAddress',
				'rules' => 'required|valid_email'
			),
			array(
				'field' => 'name',
				'label' => 'Name',
				'rules' => 'required|alpha'
			),
			array(
				'field' => 'title',
				'label' => 'Title',
				'rules' => 'required'
			),
			array(
				'field' => 'message',
				'label' => 'MessageBody',
				'rules' => 'required'
			)
		)
	);

呼叫某組驗證規則
=============================

如要呼叫特定之驗證規則，你必須在 ``run()`` 方法中傳遞該組之名字。
舉例來說，你要呼叫 signup 這組驗證規則，你可以如下這樣做::

	if ($this->form_validation->run('signup') == FALSE)
	{
		$this->load->view('myform');
	}
	else
	{
		$this->load->view('formsuccess');
	}

關聯控制器與特定驗證規則
=================================================

An alternate (and more automatic) method of calling a rule group is to
name it according to the controller class/method you intend to use it
with. For example, let's say you have a controller named Member and a
method named signup. Here's what your class might look like::

	<?php

	class Member extends CI_Controller {

		public function signup()
		{
			$this->load->library('form_validation');

			if ($this->form_validation->run() == FALSE)
			{
				$this->load->view('myform');
			}
			else
			{
				$this->load->view('formsuccess');
			}
		}
	}

In your validation config file, you will name your rule group
member/signup::

	$config = array(
		'member/signup' => array(
			array(
				'field' => 'username',
				'label' => 'Username',
				'rules' => 'required'
			),
			array(
				'field' => 'password',
				'label' => 'Password',
				'rules' => 'required'
			),
			array(
				'field' => 'passconf',
				'label' => 'PasswordConfirmation',
				'rules' => 'required'
			),
			array(
				'field' => 'email',
				'label' => 'Email',
				'rules' => 'required'
			)
		)
	);

When a rule group is named identically to a controller class/method it
will be used automatically when the ``run()`` method is invoked from that
class/method.

.. _using-arrays-as-field-names:

***************************
Using Arrays as Field Names
***************************

The Form Validation class supports the use of arrays as field names.
Consider this example::

	<input type="text" name="options[]" value="" size="50" />

If you do use an array as a field name, you must use the EXACT array
name in the :ref:`Helper Functions <helper-functions>` that require the
field name, and as your Validation Rule field name.

For example, to set a rule for the above field you would use::

	$this->form_validation->set_rules('options[]', 'Options', 'required');

Or, to show an error for the above field you would use::

	<?php echo form_error('options[]'); ?>

Or to re-populate the field you would use::

	<input type="text" name="options[]" value="<?php echo set_value('options[]'); ?>" size="50" />

You can use multidimensional arrays as field names as well. For example::

	<input type="text" name="options[size]" value="" size="50" />

Or even::

	<input type="text" name="sports[nba][basketball]" value="" size="50" />

As with our first example, you must use the exact array name in the
helper functions::

	<?php echo form_error('sports[nba][basketball]'); ?>

If you are using checkboxes (or other fields) that have multiple
options, don't forget to leave an empty bracket after each option, so
that all selections will be added to the POST array::

	<input type="checkbox" name="options[]" value="red" />
	<input type="checkbox" name="options[]" value="blue" />
	<input type="checkbox" name="options[]" value="green" />

Or if you use a multidimensional array::

	<input type="checkbox" name="options[color][]" value="red" />
	<input type="checkbox" name="options[color][]" value="blue" />
	<input type="checkbox" name="options[color][]" value="green" />

When you use a helper function you'll include the bracket as well::

	<?php echo form_error('options[color][]'); ?>


**************
Rule Reference
**************

The following is a list of all the native rules that are available to
use:

========================= ========== ============================================================================================= =======================
Rule                      Parameter  Description                                                                                   Example
========================= ========== ============================================================================================= =======================
**required**              No         Returns FALSE if the form element is empty.
**matches**               Yes        Returns FALSE if the form element does not match the one in the parameter.                    matches[form_item]
**regex_match**           Yes        Returns FALSE if the form element does not match the regular expression.                      regex_match[/regex/]
**differs**               Yes        Returns FALSE if the form element does not differ from the one in the parameter.              differs[form_item]
**is_unique**             Yes        Returns FALSE if the form element is not unique to the table and field name in the            is_unique[table.field]
                                     parameter. Note: This rule requires :doc:`Query Builder <../database/query_builder>` to be
                                     enabled in order to work.
**min_length**            Yes        Returns FALSE if the form element is shorter than the parameter value.                        min_length[3]
**max_length**            Yes        Returns FALSE if the form element is longer than the parameter value.                         max_length[12]
**exact_length**          Yes        Returns FALSE if the form element is not exactly the parameter value.                         exact_length[8]
**greater_than**          Yes        Returns FALSE if the form element is less than or equal to the parameter value or not         greater_than[8]
                                     numeric.
**greater_than_equal_to** Yes        Returns FALSE if the form element is less than the parameter value,                           greater_than_equal_to[8]
                                     or not numeric.
**less_than**             Yes        Returns FALSE if the form element is greater than or equal to the parameter value or          less_than[8]
                                     not numeric.
**less_than_equal_to**    Yes        Returns FALSE if the form element is greater than the parameter value,                        less_than_equal_to[8]
                                     or not numeric.
**in_list**               Yes        Returns FALSE if the form element is not within a predetermined list.                         in_list[red,blue,green]
**alpha**                 No         Returns FALSE if the form element contains anything other than alphabetical characters.
**alpha_numeric**         No         Returns FALSE if the form element contains anything other than alpha-numeric characters.
**alpha_numeric_spaces**  No         Returns FALSE if the form element contains anything other than alpha-numeric characters
                                     or spaces.  Should be used after trim to avoid spaces at the beginning or end.
**alpha_dash**            No         Returns FALSE if the form element contains anything other than alpha-numeric characters,
                                     underscores or dashes.
**numeric**               No         Returns FALSE if the form element contains anything other than numeric characters.
**integer**               No         Returns FALSE if the form element contains anything other than an integer.
**decimal**               No         Returns FALSE if the form element contains anything other than a decimal number.
**is_natural**            No         Returns FALSE if the form element contains anything other than a natural number:
                                     0, 1, 2, 3, etc.
**is_natural_no_zero**    No         Returns FALSE if the form element contains anything other than a natural
                                     number, but not zero: 1, 2, 3, etc.
**valid_url**             No         Returns FALSE if the form element does not contain a valid URL.
**valid_email**           No         Returns FALSE if the form element does not contain a valid email address.
**valid_emails**          No         Returns FALSE if any value provided in a comma separated list is not a valid email.
**valid_ip**              No         Returns FALSE if the supplied IP is not valid.
                                     Accepts an optional parameter of 'ipv4' or 'ipv6' to specify an IP format.
**valid_base64**          No         Returns FALSE if the supplied string contains anything other than valid Base64 characters.
========================= ========== ============================================================================================= =======================

.. note:: These rules can also be called as discrete methods. For
	example::

		$this->form_validation->required($string);

.. note:: You can also use any native PHP functions that permit up
	to two parameters, where at least one is required (to pass
	the field data).

******************
Prepping Reference
******************

The following is a list of all the prepping methods that are available
to use:

==================== ========= =======================================================================================================
Name                 Parameter Description
==================== ========= =======================================================================================================
**prep_for_form**    No        Converts special characters so that HTML data can be shown in a form field without breaking it.
**prep_url**         No        Adds "\http://" to URLs if missing.
**strip_image_tags** No        Strips the HTML from image tags leaving the raw URL.
**encode_php_tags**  No        Converts PHP tags to entities.
==================== ========= =======================================================================================================

.. note:: You can also use any native PHP functions that permits one
	parameter, like ``trim()``, ``htmlspecialchars()``, ``urldecode()``,
	etc.

.. _class-reference:

***************
Class Reference
***************

.. php:class:: CI_Form_validation

	.. php:method:: set_rules($field[, $label = ''[, $rules = '']])

		:param	string	$field: Field name
		:param	string	$label: Field label
		:param	mixed	$rules: Validation rules, as a string list separated by a pipe "|", or as an array or rules
		:returns:	CI_Form_validation instance (method chaining)
		:rtype:	CI_Form_validation

		Permits you to set validation rules, as described in the tutorial
		sections above:

		-  :ref:`setting-validation-rules`
		-  :ref:`saving-groups`

	.. php:method:: run([$group = ''])

		:param	string	$group: The name of the validation group to run
		:returns:	TRUE on success, FALSE if validation failed
		:rtype:	bool

		Runs the validation routines. Returns boolean TRUE on success and FALSE
		on failure. You can optionally pass the name of the validation group via
		the method, as described in: :ref:`saving-groups`

	.. php:method:: set_message($lang[, $val = ''])

		:param	string	$lang: The rule the message is for
		:param	string	$val: The message
		:returns:	CI_Form_validation instance (method chaining)
		:rtype:	CI_Form_validation

		Permits you to set custom error messages. See :ref:`setting-error-messages`

	.. php:method:: set_error_delimiters([$prefix = '<p>'[, $suffix = '</p>']])

		:param	string	$prefix: Error message prefix
		:param	string	$suffix: Error message suffix
		:returns:	CI_Form_validation instance (method chaining)
		:rtype:	CI_Form_validation

		Sets the default prefix and suffix for error messages.

	.. php:method:: set_data($data)

		:param	array	$data: Array of data validate
		:returns:	CI_Form_validation instance (method chaining)
		:rtype:	CI_Form_validation

		Permits you to set an array for validation, instead of using the default
		``$_POST`` array.

	.. php:method:: reset_validation()

		:returns:	CI_Form_validation instance (method chaining)
		:rtype:	CI_Form_validation

		Permits you to reset the validation when you validate more than one array.
		This method should be called before validating each new array.

	.. php:method:: error_array()

		:returns:	Array of error messages
		:rtype:	array

		Returns the error messages as an array.

	.. php:method:: error_string([$prefix = ''[, $suffix = '']])

		:param	string	$prefix: Error message prefix
		:param	string	$suffix: Error message suffix
		:returns:	Error messages as a string
		:rtype:	string

		Returns all error messages (as returned from error_array()) formatted as a
		string and separated by a newline character.

	.. php:method:: error($field[, $prefix = ''[, $suffix = '']])

		:param	string $field: Field name
		:param	string $prefix: Optional prefix
		:param	string $suffix: Optional suffix
		:returns:	Error message string
		:rtype:	string

		Returns the error message for a specific field, optionally adding a
		prefix and/or suffix to it (usually HTML tags).

	.. php:method:: has_rule($field)

		:param	string	$field: Field name
		:returns:	TRUE if the field has rules set, FALSE if not
		:rtype:	bool

		Checks to see if there is a rule set for the specified field.

.. _helper-functions:

****************
Helper Reference
****************

Please refer to the :doc:`Form Helper <../helpers/form_helper>` manual for
the following functions:

-  :php:func:`form_error()`
-  :php:func:`validation_errors()`
-  :php:func:`set_value()`
-  :php:func:`set_select()`
-  :php:func:`set_checkbox()`
-  :php:func:`set_radio()`

Note that these are procedural functions, so they **do not** require you
to prepend them with ``$this->form_validation``.
