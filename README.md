# ProcessWire FormBuilderHtmx

A zero-configuration drop in module to power your ProcessWire FormBuilder forms with AJAX via HTMX.

Features:

- Converts any form built using the Pro FormBuilder module to AJAX
- Forms are processed in place, no page refreshes after submission
- Does not conflict with existing styles or JavaScript
- Complements FormBuilder with no modifications, all module behavior and features remain the same
- Can be used per-form alongside FormBuilder native output methods
- Compatible with [ProCache](https://processwire.com/store/pro-cache/)
- Compatible with [FieldtypeFormSelect](https://github.com/SkyLundy/FieldtypeFormSelect)

## Requirements

- [ProcessWire](https://processwire.com/) >= 3.0
- [FormBuilder](https://processwire.com/store/form-builder/) >= 0.5.5 (untested with other versions but should work)
- PHP >= 8.1
- The HTMX library **(not provided with this module)**

Ensure that HTMX is present and loaded. [See HTMX docs for details.](https://htmx.org/docs/#installing)

## How To Use

Step 1. Disable CSRF protection for the form. (see why below)

Step 2. Then...

```html
<!-- Replace $forms->render('your_form'); with this: -->
$htmxForms->render('your_form');
```

Step 3. (there is no step 3)

Ta-dah.

The `Submit` button is automatically disabled on submission to prevent duplicate requests from click-happy users.

FormBuilderHtmx uses FormBuilder's "Option C: Preferred Method" for rendering. Refer to the 'Embed' tab of your Form Setup page for additional details.

`$htmxForms->render()` is a drop-in replacement for the equivalent FormBuilder method. It can be hooked (details below) and accepts its second `$vars` argument.

### Including An Activity Indicator

Unlike a standard form which triggers a page refresh, an AJAX powered form does not provide feedback to the user that there is anything happening after taking action. FormBuilderHtmx lets you take care of that by showing the user an element of your choice. [Check out these animations for inspiration and ready-to-go code](https://cssloaders.github.io/).

Here's an example of a full implementation with a 'spinner':

```html
<style>
  /*
   * Include some CSS. The `.activity-indicator` class name is up to you. `.htmx-request` is
   * required and must remain unchanged
   */

  .activity-indicator {
    display: none;
  }

  /* Style as you please */
  .htmx-request .activity-indicator,
  .htmx-request.activity-indicator {
    display: inline;
  }
</style>

<!-- The third argument is a CSS selector matching your 'spinner' -->
<?= $htmxForms->render('your_form', [], '#your-form-indicator') ?>

<div id="your-form-indicator" class="activity-indicator">
  <span class="spinner"></span>
</div>
```

Even more ta-dah.

## CSRF Protection
**CSRF protection must be disabled for forms using HTMX/AJAX**
ProcessWire doesn't match the submission to the user so CSRF errors will occur.

## How Does It Work?

FormBuilderHtmx modfies the FormBuilder markup on page load before rendering by setting/adding attributes to the form that enable HTMX to handle submissions.

When a form is submitted, FormBuilder handles processing the data as usual and returns the full page markup. FormBuilderHtmx parses it and extracts the content HTMX expects in response.

## Hooking

FormBuilderHtmx provides a hookable method to work with the markup being output to the page after a form has been processed by FormBuilder. This works exactly as the native hookable FormBuilder::render() method does.

The markup that is passed to this hook is the markup that will be rendered to the page after being processed by FormBuilderHtmx as described above.

```php
$wire->addHookAfter('FormBuilderHtmx::render', function(HookEvent $event) {
  $formHtmlMarkup = $event->return;

  // Modify markup as desired
  $event->return =<<<EOT
    <p>Look at these beautiful AJAX processed results:</p>
    {$formHtmlMarkup}
  EOT;
});
```