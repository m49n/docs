---
subtitle: Learn how to translate content using multiple sites
---
# Multisite & Localization

The multisite feature lets you manage multiple websites from a single October CMS installation and assign content dependent on the domain name. For example, an e-commerce store with different country-specific sub-sites. You can also use it to manage translations for a localized website.

## Managing Sites

You can create sites by visiting the admin panel's **Settings → Manage Sites** page. Every installation includes a primary site that acts as the default site for every request.

The following configuration defines each site:

- **Enabled** - determines if you can access the site on the front end and admin panel.
- **Name** - specifies the site name.
- **Unique Code** - specifies a unique site code for lookup using APIs.
- **Theme** - the CMS theme to use when this site is active.
- **Locale** - the application locale used for this site.
- **Timezone** - the timezone to use for this site.
- **Custom application URL** - the URL to use when this site is active, for things like mail templates or when the link policy forces the URL.
- **Use a CMS route prefix** - defines a prefix to include before every page route. It can identify this site when using a shared hostname.
- **Define matching hostnames** - restricts the site to specific hostnames. You should enable this for production sites for security reasons. Wildcard values are supported, e.g.: `*.mydomain.tld`.
- **Display a color for this site** - when enabled, displays a banner at the top of the admin panel. Useful to identify the active site or distinguish different environments, for example, red for development, green for staging and no color for production.

When you create more than one site, each can be selected in the admin panel using the site selection dropdown menu.

## Site Picker Component

The `sitePicker` component lets you manage links to other sites. The best place to include this is in your page or layout template.

```twig
[sitePicker]
==
{% set availableSites = sitePicker.sites %}
```

The following is an example of displaying a dropdown that switches between sites. It is used in conjunction with the `this.site` [Twig property](../../markup/property/this-site.md).

```twig
<select class="form-control" onchange="window.location.assign(this.value)">
    {% for site in sitePicker.sites %}
        <option value="{{ site.url }}" {{ this.site.code == site.code ? 'selected' }}>{{ site.name }}</option>
    {% endfor %}
</select>
```

Another example is generating alternative page links using meta tags

```twig
{% for site in sitePicker.sites %}
    <link rel="alternate" hreflang="{{ site.locale }}" href="{{ site.url }}" />
{% endfor %}
```

## Browser Language Detection

October CMS is configured to automatically detect a matching site based on the browser's preferred language when the following conditions are met.

- The primary site has a CMS route prefix set
- There are no other sites matching the base URL

For example, if the primary site is **English** with a route prefix of `/en` and another site **French** has a prefix of `/fr` then the following behavior can be observed.

Base URL | Behavior
-------- | --------
https://yoursite.tld/en | Displays the **English** site
https://yoursite.tld/fr | Displays the **French** site
https://yoursite.tld | Redirect based on language preference

When the user visits the base URL, their preferred language is automatically detected and will be redirected to a matching site. The matching site is based on its locale value and if no match is found then the primary site is used.