# findthemasks.com (used by getusppe.org)

This repo hosts the code for findthemasks.com, which is also used to power the v1 backend of "give ppe" features on getusppe.org

- Stats: <https://findthemasks.com/stats.html>

## New volunteer?

Join the slack! <https://join.slack.com/t/findthemasks/shared_invite/zt-czdjjznp-p8~9oKuXtV_gn7wEBZGGoA>

- new dev? please look at issues and comment on something to grab it!
    - Check out the [Getting Started](getting_started.md) doc
- new data moderator? Join the slack and come to #data!
- not either? The most useful contribution is identifying more drop off locations and plugging them into the form linked on the public website, so if you don't see an issue here that calls to you, please work on that! Advice on making calls is in [#131](https://github.com/r-pop/findthemasks/issues/131#issuecomment-602746963)

## Current setup

- The website reads from a google sheet, generates a json blob, which is used to generate static HTML.

## Reading our data to build your own frontend

- Our data file updates every five minutes and can be read from findthemasks.com/data.json.
- If you read the json directly, you need to ignore entries without an 'x' in the first field. Otherwise, you may publish info hospitals asked to have taken down. Don't do it!
- If this sounds like too much work, then please use our:

## Embeddable widget of donation sites

- We have produced an embeddable version of our map, data and filters, without the call to action that's at the top of findthemasks.com. This was designed for getusppe.org on March 22, but can be reused by anyone.
- You can view it here: <https://findthemasks.com/give.html>
- To embed into your site, use this html snippet:

```html
<iframe style="width: 100%; height: 800px; border: none;" src="https://findthemasks.com/give.html"></iframe>
```

- We also support state specific data views, hiding the map, and hiding the filters through query params:

```html
?state={CA/WA/NY/etc}
?hide-map={true/false}
?hide-filters={true/false}
?hide-list={true/false} (also hides filters)
?hide-search={true/false} (beta)
```

All boolean parameters default to false (unless they're in beta).

So, for state-specific pages you can now use something like:
<https://findthemasks.com/give.html?state=CA&hide-map=true&hide-filters=true>
This will return just the filtered list of donations sites in California.

**Beta features:**

Since beta features are disabled by default, you can enable them via:

```
?show-search=true
```

## Changing Location and Locales

We use a directory structure to view country-specific datasets.

For example, `/us/give.html` will filter the map to the United States and `/fr/give.html` will filter to France.

To view translated version of a country you can pass in a locale parameter. `/us/give.html?locale=fr-FR`
will show the map of the United States in French and `/fr/give.html?locale=en-US` will show the map of France in English.

To add a new country, create a subdirectory under public `/public/country_code` using the alpha-2 country code: https://www.iban.com/country-codes.
Copy a `.htaccess` file from an existing country directory into the new one. Update the form redirect rule to the correct country code
and ensure that the translated file has the country code short link so that it redirects properly.

## Directory structure

- `/public` - The client-side code for the website. Currently has some symlinks to legacy file locations.
- `/functions` - The cloud function used to generate data.json. Not needed for frontend work.

## Firebase

### Basic architecture
Firebase is used to pull data from our moderated datastore and then generate a data.json. There is
a production environment [findthemasks](https://console.firebase.google.com/project/findthemasks/overview)
and a dev environment [findthemasks-dev](https://console.firebase.google.com/project/findthemasks-dev/overview).

The setup uses cloud functions to provide http endpoints, cloud-storage to keep the generated results,
and the firebase realtime database (NOT firestore) to cache oauth tokens.

Adding an oauth token requires hitting the `/authgoogleapi?sheetid=longstring` on the
cloud-function endpoint and granting an OAuth token for a user that has access to the sheet.

### How to deploy
- Install the [firebase cli](https://firebase.google.com/docs/cli?hl=vi) for your platform.
- Do once
  - `firebase login`
  - `firebase use --add findthemasks-dev`
  - `firebase use --add findthemasks`
  - `cd functions; npm install`  # Note you need node v8 or higher. Look a [nvm](https://github.com/nvm-sh/nvm#installing-and-updating)
- Switch deployment envrionments `firebase use [findthemasks or findthemasks-dev]`
- Deploy the cloud function. `cd functions; npm run deploy`

### How to set config variables.
Secrets and configs not checked into github are specified via cloud function configs.

To set a config:
```
firebase functions:config:set findthemasks.geocode_key="some_client_id"
```

In code, this can be retrieved via:
```
functions.config().findthemasks.geocode_key
```

In get all configs:
```
firebase functions:config:get
```

The namespace can be anything. Add new configs to the `findthemasks` namespace.

### How to locally develop
Firebase comes with a local emulation environment that lets you live develop
against localhost. Since we are using firebase configs, first we have to snag
the configs from the environment. Do that with:

```
firebase functions:config:get > .runtimeconfig.json
```

Next generate a new Firebase Admin SDK private key here:
  https://console.firebase.google.com/project/findthemasks-dev/settings/serviceaccounts/adminsdk

And save it to `service_key.json`

Then start up the emulator. Note this will talk to the production firebase
database (likely okay as the firebase database is just storing oauth tokens).

```
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/service_key.json
firebase emulators:start --only functions
```

This will create localhost versions of everything. Cloud functions should be on
[http://localhost:5001](http://localhost:5001) and `console.log()` messages will
stream to the terminal.

There is also a "shell"

```firebase functions:shell```

that can be used, but running the emulator and hitting with a web browser
is often easier in our simple case.

## Thanks

- The "Face With Medical Mask" favicon is used with thanks to
   [favicon.io](https://favicon.io/emoji-favicons/face-with-medical-mask/) which
   provides pre-generated favicon packages using
   [Twemoji](https://twemoji.twitter.com/). Twemoji graphics are licensed
   [CC-BY 4.0](https://creativecommons.org/licenses/by/4.0/).
- [Feather icon collection](https://github.com/feathericons/feather), licensed under [MIT License](https://github.com/feathericons/feather/blob/master/LICENSE).
