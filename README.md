# Our Umbraco TagHelpers
A community project of C# ASP.NET TagHelpers for the Open Source CMS Umbraco


## Installing
Add the following Nuget Package to your Umbraco website project `Our.Umbraco.TagHelpers` with Visual Studio, Rider or with the dotnet CLI tool as follows when inside the directory with the .CSProj file for the Umbraco website project.

```
cd MyUmbracoProject
dotnet add package Our.Umbraco.TagHelpers
```

## Setup

With the Nuget package added you need to register the collection of TagHelpers for Razor views and partials to use them.
Browse to `/Views/_ViewImports.cshtml` in your Umbraco project and add the following line at the bottom
```cshtml
@addTagHelper *, Our.Umbraco.TagHelpers
```

## `<our-dictionary>`
This is a tag helper element `<our-dictionary>` that will use the current page's request Language/Culture to use a dictionary translation from the Umbraco translation section.

* Find dictionary key
* Find translation for Current Culture/Language of the page
* If no translation found see if we have fallback attribute set fallback-lang or umb-dictionary-fallback-lang
* Attempt to find dictionary item for that ISO language fallback
* No translation found - leave default/value inside tag untouched for final fallback

```cshtml
<h3><our-dictionary key="home">My Header</our-dictionary></h3>
<h3><our-dictionary key="home" fallback-lang="da-DK">My Header</our-dictionary></h3>
```

## `our-if`
This is a tag helper attribute that can be applied to any DOM element in the razor template or partial. It will include its element and children on the page if the expression inside the `our-if` attribute evaluates to true.

### Simple Example
```cshtml
<div our-if="(DateTime.UtcNow.Minute % 2) == 0">This will only render during <strong>even</strong> minutes.</div>
<div our-if="(DateTime.UtcNow.Minute % 2) == 1">This will only render during <strong>odd</strong> minutes.</div>
```

### Example Before
```cshtml
@if (Model.ContentPickerThing != null)
{
    <a href="Model.ContentPickerThing.Url()" class="btn btn-action">
        <span>@Model.ContentPickerThing.Name</span>

        @if (Model.LinkMediaPicker != null)
        {
            <img src="@Model.LinkMediaPicker.Url()" class="img-circle" />
        }
    </a>
}
```

### Example After
```cshtml
<a our-if="Model.ContentPickerThing != null" href="@Model.ContentPickerThing?.Url()" class="btn btn-action">
    <span>@Model.ContentPickerThing.Name</span>
    <img our-if="Model.LinkMediaPicker != null" src="@Model.LinkMediaPicker?.Url()" class="img-circle" />
</a>
```

## `<our-macro>`
This tag helper element `<our-macro>` will render an Umbraco Macro Partial View and will use the current page/request for the Macro rendering & context.

If you wish, you can modify this behaviour and pass the context/content node that the Macro will render with using an optional attribute `content` on the `<our-macro>` tag and passing an `IPublishedContent` into the attribute. This allows the same Macro Partial View Macro code/snippet to work in various scenarios when the content node/context is changed.

Additionally custom Macro Parameters that can be passed through and consumed by Macro Partial Views are specified in the following way. The key/alias of the Macro Parameter must be prefixed with the following `bind:`

So to pass/set a value for the macro parameter `startNodeId` then I will need to set an attribute on the element as follows `bind:startNodeId`

```cshtml
<our-macro alias="ListChildrenFromCurrentPage" />
<our-macro alias="ListChildrenFromCurrentPage" Content="Model" />
<our-macro alias="ListChildrenFromCurrentPage" Content="Model.FirstChild()" />
<our-macro alias="ChildPagesFromStartNode" bind:startNodeId="umb://document/a878d58b392040e6ae9432533ac66ad9" />
```

## BeginUmbracoForm Replacement
This is to make it easier to create a HTML `<form>` that uses an Umbraco SurfaceController and would be an alternative of using the `@Html.BeginUmbracoForm` approach. This taghelper runs against the `<form>` element along with these attributes `our-controller`and `our-action` to help generate a hidden input field of `ufprt` containing the encoded path that this form needs to route to.

https://our.umbraco.com/Documentation/Fundamentals/Code/Creating-Forms/

### Before
```cshtml
@using (Html.BeginUmbracoForm("ContactForm", "Submit", FormMethod.Post, new { @id ="customerForm", @class = "needs-validation", @novalidate = "novalidate" }))
{
    @Html.ValidationSummary()

    <div class="input-group">
        <p>Name:</p>
        @Html.TextBoxFor(m => m.Name)
        @Html.ValidationMessageFor(m => m.Name)
    </div>
    <div>
        <p>Email:</p>
        @Html.TextBoxFor(m => m.Email)
        @Html.ValidationMessageFor(m => m.Email)
    </div>
    <div>
        <p>Message:</p>
        @Html.TextAreaFor(m => m.Message)
        @Html.ValidationMessageFor(m => m.Message)
    </div>
    <br/>
    <input type="submit" name="Submit" value="Submit" />
}
```

### After
```cshtml
<form our-controller="ContactForm" our-action="Submit" method="post" id="customerForm" class="fneeds-validation" novalidate>
    <div asp-validation-summary="All"></div>

    <div class="input-group">
        <p>Name:</p>
        <input asp-for="Name" />
        <span asp-validation-for="Name"></span>
    </div>
    <div>
        <p>Email:</p>
        <input asp-for="Email" />
        <span asp-validation-for="Email"></span>
    </div>
    <div>
        <p>Message:</p>
        <textarea asp-for="Message"></textarea>
        <span asp-validation-for="Message"></span>
    </div>
    <br/>
    <input type="submit" name="Submit" value="Submit" />
</form>
```

## `<our-lang-switcher>`
This tag helper element `<our-lang-switcher>` will create a simple unordered list of all languages and domains, in order to create a simple language switcher.
As this produces alot of HTML markup that is opionated with certain class names and elements, you may wish to change and control the markup it produces. 

With this tag helper the child DOM elements inside the `<our-lang-switcher>` element is used as a Mustache templating language to control the markup.

### Example
```cshtml
<our-lang-switcher>
    <div class="lang-switcher">
        {{#Languages}}
            <div class="lang-switcher__item">
                <a href="{{Url}}" lang="{{Culture}}" hreflang="{{Culture}}" class="lang-switcher__link {{#IsCurrentLang}}selected{{/IsCurrentLang}}">{{Name}}</a>
            </div>
        {{/Languages}}
    </div>
</our-lang-switcher>

<div class="lang-switcher">
    <div class="lang-switcher__item">
         <a href="https://localhost:44331/en" lang="en-US" hreflang="en-US" class="lang-switcher__link selected">English</a>
     </div>
    <div class="lang-switcher__item">
         <a href="https://localhost:44331/dk" lang="da-DK" hreflang="da-DK" class="lang-switcher__link ">dansk</a>
     </div>
</div>
```

If you do not specify a template and use `<our-lang-switcher />` it will use the following Mustache template
```mustache
<ul class='lang-switcher'>
    {{#Languages}}
        <li>
            <a href='{{Url}}' lang='{{Culture}}' hreflang='{{Culture}}' class='{{#IsCurrentLang}}selected{{/IsCurrentLang}}'>
                {{Name}}
            </a>
        </li>
    {{/Languages}}
</ul>
```


## `<our-svg>`
This tag helper element `<our-svg>` will read the file contents of an SVG file and output it as an inline SVG in the DOM.
It can be used in one of two ways, either by specifying the `src` attribute to a physcial static file served from wwwRoot or by specifying the `media-item` attribute to use a picked IPublishedContent Media Item.

```cshtml
<our-svg src="/assets/icon.svg" />
<our-svg media-item="@Model.Logo" />
```

## `<our-fallback>`
This tag helper element `<our-fallback>` uses the same fallback mode logic that is only available on the `Value()` method of the `IPublishedContent` interface that uses a string for the property name to lookup. In addition if the fallback value from a language or ancestors is not available we are still able to fallback to the content inside the tag.

```cshtml
@* Current way *@
@Model.Value("Header", fallback:Fallback.ToLanguage)

<h3><our-fallback property="Header" mode="Fallback.ToLanguage" culture="da-DK">I do NOT have a DK culture variant of this property</our-fallback></h3>
<h3><our-fallback property="Header" mode="Fallback.ToAncestors">I do NOT have a Header property set on ANY parent and ancestors</our-fallback></h3> 
```

## `<our-version>`
This tag helper element `<our-version>` prints out version number for a given Assembly name loaded into the current AppDomain or if none is given then the EntryAssembly version is displayed, which would be the Umbraco website project you are building.

```cshtml
<!-- Prints out the Website Project Assembly Version -->
<our-version />

<!-- Prints out the version number of a specific assembly loaded into Current AppDomain -->
<our-version="Our.Umbraco.TagHelpers" />
```

## `our-member-include` and `our-member-exclude`
This is a tag helper attribute that can be applied to any DOM element in the razor template or partial. It will show or hide its element and children on the page when passing a comma seperated string of member groups that the current logged in member for the exclude or include variants.

There are two special Member Groups you can use:
* `*` - All anonymous users 
* `?` - All authenticated users

```cshtml
<div our-member-include="Staff">Only members of Staff Member Group will see this.</div>
<div our-member-include="Staff,Admins">Only members of Staff OR Admins member group will see this.</div>
<div our-member-include="*">Only logged in members will see this.</div>
<div our-member-include="?">Only anonymous members will see this.</div>

<div our-member-exclude="Staff">Only Staff members can't see this (Including anonymous).</div>
<div our-member-exclude="?">Everyone except Anonymous members will see this.</div>
<div our-member-exclude="*">Everyone except who is authenticated will see this.</div>
```

## `<our-edit-link>`
This is a tag helper element which renders an edit link on the front end only if the current user is logged into umbraco and has access to the content section. 

The link will open the current page in the umbraco backoffice. You can override the umbraco url if you are using a different url for the backoffice.

### Simple example
This is the most basic example. The link will render wherever you put it in the HTML.

```html
<our-edit-link>Edit</our-edit-link>
```

It will output a link link this, where 1057 is the id of the current page:

```html
<a href="/umbraco#/content/content/edit/1057">Edit</a>
```

### Use Default Styles example

If you add an attribute of `use-default-styles`, it will render the link fixed to the bottom left of the screen with white text and a navy blue background.

```html
<our-edit-link use-default-styles>Edit</our-edit-link>
```

### Change the edit url

Perhaps you have changed your umbraco path to something different, you can use the `edit-url` attribute to change the umbraco edit content url:

```html
<our-edit-link edit-url="/mysecretumbracopath#/content/content/edit/">Edit</our-edit-link>
```

### Open in a new tab

As the edit link is just an `a` tag, you can add the usual attributes like `target` and `class` etc.
If you want the edit link to open in a new tab, just add the `target="_blank"` attribute.

```html
<our-edit-link target="_blank">Edit</our-edit-link>
```

## Video 📺
[![How to create ASP.NET TagHelpers for Umbraco](https://user-images.githubusercontent.com/1389894/138666925-15475216-239f-439d-b989-c67995e5df71.png)](https://www.youtube.com/watch?v=3fkDs0NwIE8)


## Attribution
The logo for Our.Umbraco.TagHelpers is made by <a href="https://www.freepik.com" title="Freepik">Freepik</a> from <a href="https://www.flaticon.com/" title="Flaticon">www.flaticon.com</a>
