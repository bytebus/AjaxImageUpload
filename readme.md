# Ajax image upload helper for Processwire 2.x

## Introduction

This module is more a proof-of-concept than a stable drop-in and it's in a really early stage with an ugly API. I still decided to publish it because it maybe includes some valuable helpers how an Ajax file upload in Processwire can be done.

## Installation

Copy the files into your `site/modules` directory and install in the backend as you're used to.
This will automatically add the needed (system) templates and a new page called "Ajax Image Upload" in your pages tree. Newly uploaded images will be stored with pages that are children of this page.

## Usage

You would want to use this inside a form that ultimatively creates a new page with an image uploaded via Ajax.
In the template of the form page you will have to add the following:

    $ajaxImageUpload = $modules->get('AjaxImageUpload');
    $ajaxImageUpload->addScripts();

This will make the module available in your template as `$ajaxImageUpload` and add the [jQuery File Upload](http://blueimp.github.io/jQuery-File-Upload/) script to `$config->scripts` (you are still obliged to include jQuery and actually load the scripts in `$config->scripts` on your own).

At some point you should then output a form field to hold your image:

    <form id="my_form">
        <label for="my_img">Image</label>
        <span class="button-image-upload">
            <span class="button-image-upload-label">Bilddatei auswählen...</span>
            <input type="file" id="my_img" name="my_img" data-url="<?php echo $config->urls->root . $ajaxImageUpload->handlerUrl; ?>" />
        </span>
    </form>

You will also have to set up jQuery File Upload as [documented](https://github.com/blueimp/jQuery-File-Upload/wiki) in your JS on the page:

    var $myForm = $('#my_form'),
      $myImgField = $myForm.find('#my_img');

    $myImgField.fileupload({
      dataType: 'json',
      done: function (e, data) {
          if (data.result.hasOwnProperty('pageName') && data.result.pageName) {
            $myForm.find('#my_image_page').remove();
            $('<input />')
              .attr('type', 'hidden')
              .attr('id', 'my_img_page')
              .attr('name', 'my_img_page')
              .val(data.result.pageName)
              .appendTo($myForm);
          }
        }
    });

You can use all their nice configuration options and callbacks to make it really do what you want, i.e. show a preview after successful upload, show a progress meter and the likes, but this should do all that is needed.
When the upload works as it should, the response should have a variable (`pageName`) that holds the name of the newly created page that holds your image. It is then added to your form as a hidden input with the name `my_img_page`.

When you want to use this image on a page, you can make use of the `transferImages` method of the module. This might look like this:

    $targetPage = new Page(); // the page you want to insert the image on
    $targetFieldname = 'my_img_field'; // the field into which you want to insert the image

    if ($input->post->my_img_page) {
        /* Sanitize the upload page name */
        $myImgPageName = $sanitizer->name($input->post->my_img_page);

        /* Get the upload page */
        $myImgPage = $pages->get("name=$myImgPageName,template=ajax_image_upload");
        
        if ($myImgPage && !($myImgPage instanceof NullPage)) {
            /* Transfer image from upload page to target page */
            $ajaxImageUpload->transferImages($myImgPage, $targetPage, $targetFieldname);
        }
        /* At some point save the target page */
        $targetPage->save();
    }

At this point, your image uploaded via ajax should be part of the target page.

## To do

* A much, much, much cleaner and DRYer API
* Ready to use client-side code
* Proper documentation