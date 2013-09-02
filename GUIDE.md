# Forms in Yii

In [Yii][yii], it allows you to build forms with validation quickly and easily. A quick tutorial can be found in the
[definitive guide][guide] for making [form models][form.model], [what to put in your controller actions][form.action],
and how to [display forms in a view][form.view].

This is only the first learning curve in getting to grips with forms. The correct way is to incorporate all of these
methods into the [Form Builder][form.builder], provided by Yii through the [`CForm`][cform] class.

Once completely mastered, it provides [separation of concerns][soc] which has many benefits, including:

* Use the same form model for many forms, where a different set of form inputs use the same validation rules (for
  example, login form and a registration form use the same validation rules).
* Use the same form for many form models, where the same information is needed for each model (for example, updating a
  vehicle needs the same information as creating a vehicle).
* Use the same form on multiple models, where information is divided (for example, collect information about a customer
  and their vehicle, dividing the information between the Customer and Vehicle models).
* Complete form logic is split over four files: actions, form configuration, form models, and views.
  * Actions only need to concern themselves about *what* to do with the data.
  * Form configuration defines what the input types are (text, dropdown, radio buttons, checkboxes, etc), default
    values, whether they apply to this scenario, etc.
  * Form models only need to concern whether the data passes validation, if it doesn't it will define the errors for
    each input but does not have to concern about how they are displayed.<br />For example, `input1` pass, `input2` does
    not pass with error message *X*, `input3` pass.
  * Views only have to concern themselves about *how to display* inputs. It does not matter to the view what type of
    input, whether it validates, or what to do with the data. An input is an input, they only need to worry about where
    to put it and what it looks like.<br />
    For example: display label, input and - if one has been defined - error message.
* Changing the code of one does not affect the others:
  * Changing the way `input1` gets stored in the database inside the action does not stop the input being the type it
    is, passing validation checks, or being displayed.
  * Changing the configuration of `input1` from text to dropdown does not stop its value passing validation, how it is
    stored in the database, or how it is displayed on the page (to a certain extent; an input is still an input within a
    HTML form).
  * Changing the validation of `input1` from a maximum of 10 characters to a maximum of 8 characters does change the
    type of input it is, how it is displayed, nor does it stop the action from recieving the value of the input.
  * And of course, changing how you display it to the end-user in the view does not alter its validation rules, what
    type of input it is, or what the action does with its value.
* Separating the type of input into the form configuration and away from the view means that the input type can be
  changed without having to change the HTML tag in each theme that uses that form. one of the most useful features.

> **Please note:** [`CForm`][cform] is a component and can be extended, just like any other in Yii. To provide the extra
> functionality explained in this document, [`CForm`][cform] is extended by the class `application\components\Form` -
> which is a custom class by [Zander Baldwin][zander] originally for a proprietary project. If you see a reference to
> [`CForm`][cform] please remember that it is for inheritance checks only and the actual class of the form object is
> `application\components\Form`.

## Form Models

The first part of a form is to create the model. It is a class with a public property for every user input, where the
name of the property is the name of the input. For the rest of this document we will use the login page form as an
example; so for the class definition of the form model it would look something like this:

```php
<?php

    class LoginForm extends CFormModel
    {

        /**
         * @var string $username
         */
        public $username;

        /**
         * @var string $passphrase
         */
        public $passphrase;

        /**
         * @var boolean $remember
         */
        public $remember;

    }
```

It should also define two methods: [`rules()`][form.rules] and [`attributeLabels()`][form.labels].
Both of these methods return arrays. `rules()` is for [defining validation rules][definingrules] and `attributeLabels()`
is for [overriding label names][defininglabels] for each input (which is used when displaying the form).

### Example Validation

```php
<?php

    class LoginForm extends CFormModel
    {

        /**
         * @var string $username
         */
        public $username;

        /**
         * @var string $passphrase
         */
        public $passphrase;

        /**
         * @var boolean $remember
         */
        public $remember;

        /**
         * Validation Rules
         *
         * @access public
         * @return array
         */
        public function rules()
        {
            return array(
                array('username, passphrase', 'required'),
                array('username', 'length', 'max' => 64),
                array('remember', 'boolean'),
            );
        }

    }
```

### Example Attributes

If you don't define the `attributeLabels()` method, Yii will fallback to using the name of input/property for the label.
Which is better to be displayed on the HTML page: `dob_year` or `Year of Birth`?

Don't forget to wrap your attribute labels with `Yii::t`, just like all other text that gets presented to the end-user.
You may not need it, but getting into the habit of it makes things easier when you want to automatically translate your
application into another language. Think of the extra revenue when we could not only say "Made in Wales", but also sell
the product in Welsh?

```php
<?php

    class LoginForm extends CFormModel
    {

        /**
         * @var string $username
         */
        public $username;

        /**
         * @var string $passphrase
         */
        public $passphrase;

        /**
         * @var boolean $remember
         */
        public $remember;

        /**
         * Validation Rules
         *
         * @access public
         * @return array
         */
        public function rules()
        {
            return array(
                array('username, passphrase', 'required'),
                array('username', 'length', 'max' => 64),
                array('remember', 'boolean'),
            );
        }

        /**
         * Attribute Labels
         *
         * @access public
         * @return array
         */
        public function attributeLabels()
        {
            return array(
                'username'      => Yii::t('system60', 'Username or Email'),
                'passphrase'    => Yii::t('system60', 'Pass Phrase'),
                'rememer'       => Yii::t('system60', 'Remember?')
            );
        }

    }
```

## Form Configuration

Although configuration can be directly passed to the form object, it is best to keep it in a separate file. Form
configuration files are placed in their own directory in the application, and `forms` seems like a good choice.

> **Please note:**
> In the Nosco Systems project, `application.forms` is used to store form models.
>
> System60 stores form models in the directory `application.models.form` instead so that `application.forms` can be used
> to store form configuration files.

A configuration file is just like the application configuration file. It is a PHP script that returns an array of
configuration settings relating the properties of the class it is designed for; in this case [`CForm`][cform]
(`application\components\Form` does not have any configurable properties). These properties are:

* [`action`](http://yiiframework.com/doc/api/CForm#action-detail "CForm::action"): the URL that the form should submit
  to. If one is not supplied, it defaults to the URL of the page that the form is being displayed on.
* [`attributes`](http://yiiframework.com/doc/api/CForm#attributes-detail "CForm::attributes"): any extra HTML attributes
  that the `<form>` tag should have. You should refrain from setting this in the configuration and leave it until inside
  the view, as different themes may want different HTML attributes.
* [`description`](http://yiiframework.com/doc/api/CForm#description-detail "CForm::description"): an optional string
  description for the form.
* [`method`](http://yiiframework.com/doc/api/CForm#method-detail "CForm::method"): the submission method of the form
  (`get` or `post`, defaults to `post`).
* [`model`](http://yiiframework.com/doc/api/CForm#model-detail "CForm::model"): the instance of the model that is
  associated with this form. This shouldn't be used as it is passed in the second parameter of the form object.
* [`title`](http://yiiframework.com/doc/api/CForm#title-detail "CForm::title"): an optional string title for the form.
  This is usually used with the fieldset legend, but can be customised in the view.

The most important properties, however, are:

* [`elements`]( "CForm::elements"): configures the input elements of this form.
* [`buttons`]( "CForm::buttons"): configures the button elements of this form.

### Input Elements

Each input element can be three different things: a string, a sub-form nested inside the current one, and obviously an
input element. Strings are obsolete when forms are customised by views, and this document does not cover sub-forms.

In the `elements` array, the array key is the name of the input which must match a property set in the form model -
otherwise the input element would not be able to be paired with any defined validation rules. The array value is another
array containing the following keys:

* [`hint`](http://www.yiiframework.com/doc/api/1.1/CFormInputElement#hint-detail "CFormInputElement::hint"): an optional
  hint to display next to the input to help the end-user understand the requirements of the input.
* [`items`](http://www.yiiframework.com/doc/api/1.1/CFormInputElement#items-detail "CFormInputElement::items"): an array
  of items, where the array key is the value to be submitted in the form, and the array value is the text to display.
* [`label`](http://www.yiiframework.com/doc/api/1.1/CFormInputElement#label-detail "CFormInputElement::label"): an
  optional string used to override the value set in the form model.
* [`type`](http://www.yiiframework.com/doc/api/1.1/CFormInputElement#type-detail "CFormInputElement::type"): the type of
  input this element is. It can be any of the following:
  * `text`.
  * `hidden`.
  * `password`.
  * `textarea`.
  * `file`.
  * `radio`.
  * `checkbox`.
  * `listbox`.
  * `dropdownlist`.
  * `checkboxlist`.
  * `radiolist`.
  * There are also five options for HTML5 inputs: `url`, `email`, `number`, `range`, and `date`. These are not advised
    as they will not work in any browser that does not support the HTML5 Input API.

Any other array key for each element that does not correspond to a property of
[`CFormInputElement`](http://yiiframework.com/doc/api/CFormButtonElement "CFormInputElement") becomes a HTML attribute
of the element. For example, an array key of `maxlength` for a text input element will limit the amount of characters
allowed inside that element.

#### List Items

If the input type contains the word "list", then the `items` property needs to be set: `listbox`, `dropdownlist`,
`checkboxlist` and `radiolist` won't work if they don't have a list of options for the end-user to choose from.

### Button Elements

Button elements are exactly the same as input elements except that they only have two properties:

* [`label`](http://www.yiiframework.com/doc/api/1.1/CFormButtonElement#label-detail "CFormButtonElement::label"): a
  string defining the text that the button shows. If this is not set it, the browser will automatically set this
  (usually to "Submit").
* [`type`](http://www.yiiframework.com/doc/api/1.1/CFormButtonElement#type-detail "CFormButtonElement::type"): the type
  of button this element represents. It can be any of the following:
  * `htmlButton`.
  * `htmlReset`.
  * `htmlSubmit`.
  * `submit`: this is the most common for forms.
  * `button`.
  * `image`: if this type is chosen, it is advisable to provide a `src` property too.
  * `reset`.
  * `link`.

### Example Form Configuration

```php
<?php

    return array(
        'title' => Yii::t('system60', 'Login'),
        'description' => Yii::t(
            'system60',
            'Fields marked with {asterix} are required.',
            array('{asterix}' => '<span class="required">*</span>')
        ),
        'method' => 'post',

        'elements' => array(
            'username' => array(
                'type' => 'text',
                'maxlength' => 64,
                'hint' => Yii::t('system60', 'Please enter your username; it is case-insensitive.'),
            ),
            'passphrase' => array(
                'type' => 'password',
                'hint' => Yii::t('system60', 'Please enter your password; it is case-sensistive.'),
            ),
            'remember' => array(
                'type' => 'checkbox',
                'hint' => Yii::t('system60', 'Remember who you are to skip login screen next time?'),
            ),

            // This one isn't a real input element for our form, just an example to show lists.
            'exampleList' => array(
                'type' => 'dropdownlist',
                // Would be much easier to pull options from a model, by the way. Hint hint.
                'items' => array(
                    'value1' => Yii::t('system60', 'First Option'),
                    'value2' => Yii::t('system60', 'Second Option'),
                    'value3' => Yii::t('system60', 'Third Option'),
                ),
                'prompt' => Yii::t('system60', 'Please select an option...'),
            ),

        ),

        'buttons' => array(
            'submit' => array(
                'type' => 'submit',
                'label' => Yii::t('system60', 'Login'),
            ),
        ),
    );
```

## Form Actions

```php
<?php
    // Create an instance of the form model, this controls all of the validation rules (data).
    $model = new YourFormModel;
    // Create an instance of the form builder, this controls all of the display logic (inputs).
    // Load up the form configuration from the path alias, and associate it with the form model.
    $form = new \application\components\Form('application.forms.YourFormConfig', $model);
    // Check that the form has been submitted, and if has, if it passes the models validation rules.
    if($form->submitted()) {
        if($form->validate()) {

            // Hurrah! We have user input that has passed validation!
            // Whatever you want to do with your data. In this case, log the user in.
            $userIdentity = new UserIdentity($model->username, $model->password);
            if($userIdentity->authenticate()) {
                Yii::app()->user->login($userIdentity);
                $this->redirect(Yii::app()->homeUrl);
            }

        }
        else {
            // Oh no, there was an error with the validation!
        }
    }
    // Pass the form object to the view so that it can display it to the user.
    $this->render('your_view', array('form' => $form));
```

The application flow goes a little something like this.

* Create an instance of the form model.
* Create an [instance of the form object](http://yiiframework.com/doc/api/CForm#__construct-detail "CForm::__construct()"),
  specifying where to find the form configuration file and which model you wish to associate with the form.
* Check that the form [has been submitted](http://yiiframework.com/doc/api/CForm#submitted-detail "CForm::submitted()"),
  and [that it validates](http://yiiframework.com/doc/api/CForm#validate-detail "CForm::validate()").
* Send the form object [to the view](http://yiiframework.com/doc/api/CController#render-detail "CController::render()")
  to be displayed.

If you take away the comments, the example data usage, and combine some control structures, the total
<abbr title="Source Lines of Code">SLOC</abbr> is 3. Seriously, this is all you have to write:

```php
<?php

    $form = new \application\components\Form('application.forms.YourFormConfig', $model = new YourFormModel);
    if($form->submitted() && $form->validate()) {
        // Do your stuff.
    }
    $this->render('your_view', array('form' => $form));
```

### Some Points on Application Flow

The method `$form->submitted()` must be called *before* `$form->validate()`. This is because behind the scenes, it
checks that the form has been submitted **and** if it has, it loads the form data into the form model. The model cannot
validate the user input if it hasn't been loaded in yet.

In `CForm` model, the default functionality of `$form->submitted()` is to check that a button named "submit" has been
posted along with the form data. If you didn't have a button called "submit" the default functionality would always
return false unless you specified the name of your default button in the first parameter.<br />
In `application\components\Form`, the default functionality is to check that **any** button defined in the form
configuration has been posted along with the form data. You can then use the first parameter to check if a specific
button has been posted instead; for example, `$form->submitted('nameOfButton')` - useful when you one form with many
option buttons like "update", "delete", "duplicate", etc.

## Form Views

> Most extra functionality from `application\components\Form` applies to form views.
>
> It adds the methods `input()`, `button()` and `hint()` to the active form widget, and improves the functionality of
> the methods `label()`, `labelEx()`, `error()` and `errorSummary()` so that it accepts both form models and the form
> object.

To begin with, a simple way to display the form is to simply echo the form object. This will render your form with
**very** basic HTML. This is useful when you wish to see the progress developing the other parts of the form, but it
does mean that you have no control over the layout or style of the form.

```php
<?php echo $form; ?>
```

### Starting the Form

Before you begin the form, you may want to add some HTML attributes to the form tag, like a couple of CSS classes, which
can be done by setting [`$form->attributes`](http://yiiframework.com/doc/api/CForm#attributes-detail "CForm::attributes")
to an array. This is just a tidier way of setting the HTML options of the active form widget with
`$form->activeForm['htmlOptions']`.<br />
Then to start the form, you have to begin rendering it with the
[`renderBegin()`](http://yiiframework.com/doc/api/CForm#renderBegin-detail "CForm::renderBegin()") method, remembering
to print it to the output.<br />
Finally, you should grab a copy of the active form widget as this is what you'll use to build all the inputs and buttons
with.

```php
<?php

    $form->attributes = array(
        'class' => 'form-horizontal',
    );
    echo $form->renderBegin();
    $widget = $this->activeFormWidget();
```

> **Please note:** always use the `renderBegin()` method, instead of writing your own `<form>` tags. This is because it
> will also generate a hidden field with a unique value that changes for each user.
>
> If this hidden field is not detected or is incorrect, it assumes that the form is being spammed by bots instead of
> actual humans visiting the page; in which case it will say that the form has not been submitted so that the
> application will not have to deal with suspicious data.

### The Elements

Each element has four parts to it: the label, the input, the hint, and the error message.

#### Label

The active form widget has two methods for labels;
[`label()`](http://yiiframework.com/doc/api/CActiveForm#label-detail "CActiveForm::label()") and
[`labelEx()`](http://yiiframework.com/doc/api/CActiveForm#labelEx-detail "CActiveForm::labelEx()"). `labelEx()` provides
exactly the same functionality as `label()` with the added bonus of appending an asterix at the end of labels whose
inputs are required, so this method is preferred. Both take the same, three arguments:

* `$form`: the form object, or the form model associated with it.
* `$attribute`: the name of the input element.
* `$htmlOptions`: an array of extra HTML attributes to add to the label.

#### Input

The active form widget has many methods for inputs &mdash; one for each type to be exact &mdash; but the most useful one
is `input()` as it will automatically detect the input type and call the appropriate method to render it. It, again,
takes three arguments:

* `$form`: the form object. It does not accept form models as input types are only stored in the form configuration.
* `$attribute`: the name of the input element.
* `$htmlOptions`: an array of extra HTML attributes to add to the input.

#### Errors

The active form widget has two methods for errors;
[`error()`](http://yiiframework.com/doc/api/CActiveForm#error-detail "CActiveForm::error()") and
[`errorSummary()`](http://yiiframework.com/doc/api/CActiveForm#errorSummary-detail "CActiveForm::errorSummary()").
The former produces the error message for a specified input, whilst the latter generates a list of all errors in the form.

`error()` takes 5 arguments, but please do not bother with the last two; Yii's client scripts cause more hassle than
they're worth and should be avoided - they attempt to inject Javascript into the HTML compiled from your views just
before it gets sent to the end-user.

* `$form`: the form object, or the form model associated with it.
* `$attribute`: the name of the input element.
* `$htmlOptions`: an array of extra HTML attributes to add to the error's containing DIV element.
* `$enableAjaxValidation`: a boolean value determining whether or not to use AJAX validation.
* `$enableClientValidation`: a boolean value determining whether or not to use client validation.

`errorSummary()` takes 4 arguments.

* `$form`: the form object, or the form model associated with it.
* `$header`: raw HTML code that gets prepended to the error list.
* `$footer`: raw HTML code that gets appended to the error list.
* `$htmlOptions`: an array of extra HTML attributes to be added to the HTML div tag surrounding the error list.

#### Hint

The active form widget has a single method for input hints; `hint()`. It takes 4 arguments:

* `$form`: the form object. It does not accept form models as hints are only stored in the form configuration.
* `$attribute`: the name of the input element.
* `$tag`: the name of the HTML element to wrap the hint text in. If this is omitted (or is not a string) then the hint
  text will not be wrapped in a HTML element.
* `$htmlOptions`: an array of extra HTML attributes to add to HTML element wrapping the hint text, if there is one.

### The Buttons

The active form widget has many methods for buttons &mdash; one for each type to be exact &mdash; but the most useful one
is `button()` as it will automatically detect the button type and call the appropriate method to render it. It takes
three arguments:

* `$form`: the form object. It does not accept form models as button types are only stored in the form configuration.
* `$attribute`: the name of the button element.
* `$htmlOptions`: an array of extra HTML attributes to add to the button.

### Example View

```php
<?php
    $form->attributes = array('class' => 'form-horizontal');
    echo $form->renderBegin();
    $widget = $form->activeFormWidget;
?>
<fieldset>

    <?php if(isset($form->title) && is_string($form->title) && $form->title): ?>
        <legend><?php echo $form->title; ?></legend>
    <?php endif; ?>

    <?php if(isset($form->description) && is_string($form->description) && $form->description): ?>
        <p class="description"><?php echo $form->description; ?></p>
    <?php endif; ?>

    <?php
        echo $widget->labelEx($form, 'username');
        // This one outputs a text field, as per the form config.
        echo $widget->input($form, 'username');
        // If there is an error message, display that. If not, show the hint.
        echo $widget->error($form, 'username') ?: $widget->hint($form, 'username');
    ?>

    <?php
        echo $widget->labelEx($form, 'passphrase');
        // This one outputs a password field, as per the form config.
        echo $widget->input($form, 'passphrase');
        // If there is an error message, display that. If not, show the hint.
        echo $widget->error($form, 'passphrase') ?: $widget->hint($form, 'passphrase');
    ?>

    <?php
        echo $widget->labelEx($form, 'remember');
        // This one outputs a checkbox field, as per the form config.
        echo $widget->input($form, 'remember');
        // If there is an error message, display that. If not, show the hint.
        echo $widget->error($form, 'remember') ?: $widget->hint($form, 'remember');
    ?>

    <?php
        echo $widget->button($form, 'submit');
    ?>

</fieldset>
<?php echo $form->renderEnd(); ?>
```

[yii]:          http://yiiframework.com                                         "Yii Framework"
[guide]:        http://www.yiiframework.com/doc/guide/                          "The Definitive Guide to Yii"
[form.model]:   http://yiiframework.com/doc/guide/en/form.model                 "Creating Form Models"
[form.action]:  http://yiiframework.com/doc/guide/en/form.action                "Creating Form Logic in Actions"
[form.view]:    http://yiiframework.com/doc/guide/en/form.view                  "Creating Form Views"
[form.builder]: http://yiiframework.com/doc/guide/en/form.builder               "Yii Form Builder"
[cform]:        http://yiiframework.com/doc/api/CForm                           "Yii API - CForm"
[zander]:       https://github.com/mynameiszanders                              "Zander Baldwin on GitHub"
[form.rules]:   http://yiiframework.com/doc/api/CModel#rules-detail             "Yii API - CModel::rules()"
[form.labels]:  http://yiiframework.com/doc/api/CModel#attributeLabels-detail   "Yii API - CModel::attributeLabels()"
[definingrules]: http://yiiframework.com/doc/guide/en/form.model#declaring-validation-rules "Defining Validation Rules"
[defininglabels]: http://yiiframework.com/doc/guide/en/form.model#attribute-labels "Defining Attribute Labels"
[soc]:          http://en.wikipedia.org/wiki/Separation_of_concerns             "Wikipedia: Separation of Concerns"
