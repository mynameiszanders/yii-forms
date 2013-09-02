Yii Forms
=========

*Version: 1.0.0*

A collection of components classes for Yii forms to provide cross-theme rendering of form inputs, without having to know
the types of inputs that were defined in form configuration files.

To use, simply use `application\components\Form` rather than `CForm` when creating an instance of the form builder. You
might also want to read [the dummies guide](GUIDE.md) for building forms with the new classes. It's not great, and
that's because I hate writing documentation (source comments, baby!) and managed to knock this out in 4 hours.

If you want to rename `application\components\ActiveForm`, make sure to update the reference in
`application\components\Form::activeForm`.

## Changes

It adds the methods:

* `application\components\ActiveForm::input()`
* `application\components\ActiveForm::button()`
* `application\components\ActiveForm::hint()`

And extends the methods:

* [`CForm::submitted()`](http://yiiframework.com/doc/api/CForm#submitted-detail "CForm::submitted()")
* [`CActiveForm::label()`](http://yiiframework.com/doc/api/CActiveForm#label-detail "CActiveForm::label()")
* [`CActiveForm::labelEx()`](http://yiiframework.com/doc/api/CActiveForm#labelEx-detail "CActiveForm::labelEx()")
* [`CActiveForm::error()`](http://yiiframework.com/doc/api/CActiveForm#error-detail "CActiveForm::error()")
* [`CActiveForm::errorSummary()`](http://yiiframework.com/doc/api/CActiveForm#errorSummary-detail "CActiveForm::errorSummary()")
