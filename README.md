# Salesforce CumulusCI minimum viable build that will create a scratch org

While the `cci project init` command will throw a lot of useful empty files into a blank folder for you, and will create a fileset that supports launching a scratch org with the `cci flow run dev_org` command, it generates a lot of files you don't necessarily need right away.

Here's what I've found is actually necessary:

## 1:  Hand-create a `/.sfdx/sfdx-config.json` file

First things first:  you won't want to make it part of the "source code" for the CumulusCI project you're building, but every folder containing a CumulusCI project should have a `/.sfdx/sfdx-config.json` file in containing a single [JSON-formatted object](https://katiekodes.com/intro-xml-json-1/) that has exactly 1 property called `defaultusername` in it.  The value of `defaultusername` should be the alias that you gave a given Salesforce production org or developer org that you enabled "dev hub" functionality in when you logged the [Salesforce CLI](https://developer.salesforce.com/tools/sfdxcli), as installed on **your** computer, into that org.  You would have used the command `sfdx force:auth:web:login --setalias your-company-hub-org-nickname --instanceurl https://customdomain.my.salesforce.com/`.  You can see what you called it when you logged in by looking under the "Alias" column of the results of `sfdx auth:list`.

Here's an example `/.sfdx/sfdx-config.json` file:

```json
{
    "defaultdevhubusername": "your-company-hub-org-nickname"
}
```

If you see an entry under `sfdx auth:list` that you'd like to use but it has a blank value under "Alias," take the value under "username" and plug it into the `sfdx alias:set your-company-hub-org-nickname=the-username-you-chose@example.com`, substituting in appropriate values for "`your-company-hub-org-nickname`" and "`the-username-you-chose@example.com`," of course.

**Note:**  You need to do this by hand after downloading a copy of this codebase.  I did not include a sample in this codebase.

---

## 2:  Make sure you have org templates

Since CumulusCI expects you to be nicknaming scratch orgs `beta`, `dev`, `feature`, and `release` quite frequently, your project folder needs at least the following 4 files in it to serve as templates for what scratch orgs of these names should look like when CumulusCI spins them up:

1. `/orgs/beta.json`
1. `/orgs/dev.json`
1. `/orgs/feature.json`
1. `/orgs/release.json`

Each of these 4 files should be structured just like non-CumulusCI SFDX projects expect [a `/config/project-scratch-def.json` file](https://developer.salesforce.com/docs/atlas.en-us.sfdx_dev.meta/sfdx_dev/sfdx_dev_scratch_orgs_settings.htm) to be structured.  For each scratch-org-setup template, you'll tell CumulusCI:

1. What kind of plain-English **name** you'd like to give a scratch org spun up with this template _(the "`orgName`" property -- convention is to write the same phrase you put into `project.package.name` in the `cumulusci.yml` file, followed by a space, a hyphen, and a space, followed by the phrase "Beta Test Org," "Dev Org," "Feature Test Org," or "Release Test Org")_
1. What "`edition`" of Salesforce you'd like the scratch org to be _(e.g. "Developer" vs. "Enterprise" vs. "Partner Developer" vs. "Partner Enterprise")_
1. Whether there are any special "`features`" you need to be enabled in the scratch org, such as Experience Cloud _(example enabling both Experience Cloud and Service Cloud:  `"features": ["Communities, ServiceCloud"]`)_
1. All of the [many possible "`settings`" that an org can have](https://developer.salesforce.com/docs/atlas.en-us.api_meta.meta/api_meta/meta_settings.htm), such as:
    * How often the scratch org will [re-prompt you to log into it](https://developer.salesforce.com/docs/atlas.en-us.api_meta.meta/api_meta/meta_securitysettings.htm?q=forcerelogin#field_forcerelogin) _(`settings.securitySettings.sessionSettings.forceRelogin`)_
    * [Whether Lightning Experience is turned on](https://developer.salesforce.com/docs/atlas.en-us.api_meta.meta/api_meta/meta_lightningexperiencesettings.htm?q=enableS1DesktopEnabled) _(`settings.lightningExperienceSettingsenableS1DesktopEnabled`)_
    * Whether [Experience Cloud is enabled](https://developer.salesforce.com/docs/atlas.en-us.234.0.api_meta.meta/api_meta/meta_communitiessettings.htm?q=enableNetworksEnabled) _(`settings.communitiesSettings.enableNetworksEnabled`)_
    * etc.

**Note:**  No action needed here on your part -- I provided example files as part of the codebase.

---

## 3:  Make sure you specify which files codebase-tracking should ignore

You'll definitely want to make sure that files and folders you don't want [version-controlled and tracked with Git](https://katiekodes.com/git-brain-dump/) as "part of the codebase of your project" are listed in a `/.gitignore` file.

**Note:**  No action needed here on your part -- I provided an example file as part of the codebase.

---

## 4:  Define the folder as a CumulusCI project

The CumulusCI command-line tool refuses to do anything interesting when run within a folder unless that folder has a `/cumulusci.yml` file in it whose contents are formatted conformingly to the [YAML punctuation standard](https://katiekodes.com/intro-xml-json-yaml-book/).

At the very least, under a "`project`" property, you'll want to specify the following sub-properties of "`project`" when writing a codebase that you expect CumulusCI to be willing to spin up into a scratch org:

1. `name`:  Something that describes what the project is for, and has no spaces in it.
1. `package.name`:  Something that describes what the project is for, with spaces allowed.  _(Note that you'll often see whatever is chosen to go here reappear in the project templates under the `/orgs/` folder)_.
1. `git.default_branch`:  It can be very helpful to clarify whether the primary nickname of your Git version-control tracking of this folder is [called "main," "master," or something else entirely](https://sfconservancy.org/news/2020/jun/23/gitbranchname/).
1. `source_format`:  For brand new projects, just make this "`sfdx`."
    * You really don't want to start up a new project that uses the "metadata API" format -- that was the old annoying format for defining Salesforce configuration settings as plaintext files where every [field](https://help.salesforce.com/s/articleView?id=sf.customize_fields.htm&type=5) on an [object](https://help.salesforce.com/s/articleView?id=sf.schema_builder_elements_objects_ref.htm&type=5) was shoved into one really big plaintext file [formatted in the XML punctuation standard](https://katiekodes.com/intro-xml-json-1/), which made it nearly impossible for two people at a time to work on different projects involving the same object, even if they weren't working on the same fields as each other.

**Note:**  No action needed here on your part -- I provided an example file as part of the codebase.

---

## 5:  Define the folder as an SFDX project

The CumulusCI command-line tool refuses to do anything interesting when run within a folder unless that folder has a `/sfdx-project.json` file in it whose contents are formatted conformingly to the [JSON punctuation standard](https://katiekodes.com/intro-xml-json-1/).

At the very least, you'll need it to specify the following properties:

1. `"packageDirectories"`, with a list containing at least 1 object entry.  Each entry needs a `"path"` sub-property and one of them should have a `"default"` property set to `true`.  The most common setting is to have just 1 entry whose `"path"` value is "`"force-app"`."
1. `"namespace"`:  usually set to `null`, but put something here if you're about to bundle up your codebase into a managed package.
1. `"sourceApiVersion"`:  set to something like `"55.0"` -- make it a Salesforce Platform API version that's stable with your codebase.
  * Personally, when I'm developing a small teaching demonstration where API version doesn't really matter, I hate hard-coding an API version into the [Git-tracked](https://katiekodes.com/git-brain-dump/) files within my project folder.  I just don't like the way it looks to have them get "old" over the years.  However, you're in a season when Salesforce has started rolling out one of their 3x/year upgrades to certain test environments, but hasn't rolled it out into all production environments, CumulusCI might error out when you try to spin up a scratch org that doesn't have `"sourceApiVersion"` set -- or that includes some other repository as a dependency that doesn't.  The error would likely say:  "`Error: Could not process MDAPI response: Update of None package.xml: Error: Invalid version specified`."  So as much as I wish you could just add an "`"apiVersion"`" property to a file that you probably aren't tracking with Git such as `/.sfdx/sfdx-config.json`, to make CumulusCI happy year-round, set `"sourceApiVersion"`.

---

## 6:  Make a folder for the actual project work

For every folder you specified in a `"path"` sub-property of an item in the list under the `"packageDirectories"` property of the `/sfdx-project.json` file, you'll need to actually create that folder.

You don't need to _put_ anything into the folder yet.  You just need to _create_ it, empty, to prevent CumulusCI from erroring out when you try to spin up a scratch org, saying something like "`Error: Command exited with return code 1: ERROR running force:source:tracking:reset:  The path "force-app", specified in sfdx-project.json, does not exist. Be sure this directory is included in your project root.`"

However, [Git code tracking](https://katiekodes.com/git-brain-dump/) ignores empty files when backing up your codebase to a cloud host like GitHub, so if you're trying to share your project with the world like I'm doing here, you need to put _some_ sort of meaningless file inside of the folder(s) you just created.

As you can see, I've created an empty file in this project called "`/force-app/.gitkeep`" for this purpose.

It's perfectly fine to delete any ".gitkeep" files that exist in a project when you start filling their parent folders with real code, although it also usually doesn't hurt to leave them there.

Eventually, you'll end up with things that define the essential details your project needs to spin up new scratch orgs with inside of these folders.

For example, you might end up with `/force-app/main/default/classes/HelloWorld_TEST.cls` and `/force-app/main/default/classes/HelloWorld_TEST.cls-meta.xml` files in your project when you decide that every scratch org spun up from your project needs an Apex class named `HelloWorld_TEST` to exist in that scratch org.

**Note:**  No action needed here on your part -- I provided an example file as part of the codebase.

---

## HAVE FUN:  Validate that CumulusCI works

### Build a scratch org

To validate that the "minimum viable build" I've described here is _still_ all you need to make CumulusCI capable of spinning up a scratch org from a project, try the following:

```sh
cci flow run dev_org --org your-scratch-org-nickname-here
```

_(Substitute `beta`, `dev`, `feature`, or `release` for "`your-scratch-org-nickname-here`.")_

### Open the scratch org

Once it's finished, spinning up the scratch org, you can open it in a web browser with the following command:

```sh
cci org browser --org your-scratch-org-nickname-here
```

Or, if your computer's default web browser and scratch orgs don't get along, you can make your computer's command line give you a URL to hand-copy-and-paste into a different web browser:

```sh
cci org browser --org your-scratch-org-nickname-here --url-only
```

### Delete the scratch org

Once you're done, if you need to delete the scratch org before it naturally expires, you can run the following command:

```sh
cci org scratch_delete your-scratch-org-nickname-here
```

### Update your scratch org from changed dependencies

On the other hand, if all you've done is update the `dependencies` sub-property of `project` inside of a `cumulusci.yml` file, rather than deleting and recreating the scratch org, typically all you have to do is run the following _(which is really handy if one of the dependencies you've already let install is slow, like [NPSP](https://www.salesforce.org/products/nonprofit-success-pack/))_:

```sh
cci task run update_dependencies --org your-scratch-org-nickname-here
```

Head back over to the running scratch org in your web browser and reload an appropriate page in Setup to see that your changes took effect.

### Update your scratch org from your codebase

And if all you've done is hand-write some new code into `/force-app/` or wherever it is you're putting your codebase, then you don't need to delete and recreate the scratch org.  You can just run this command:

```sh
cci task run dx_push --org your-scratch-org-nickname-here
```

Head back over to the running scratch org in your web browser and reload an appropriate page in Setup to see that your changes took effect.

### Update your codebase from your scratch org

That said, usually you'll be doing things the other way around:  clicking through the browser in your scratch org, reconfiguring Salesforce, and then pulling down text-based copies of your new-and-improved configuration into your project folder with the following commands:

```sh
cci task run list_changes --org your-scratch-org-nickname-here
```

```sh
cci task run retrieve_changes --org your-scratch-org-nickname-here
```

---

## A note about dependencies

As long as you don't start playing with GitHub "releases" or Salesforce "packaging," you can include any project hosted on GitHub.com that conforms to this "minimum viable build" file-and-folder structure inside of a _different_ CumulusCI project that _also_ conforms to this "minimum viable build" structure.

All you have to do, in the second project, is reference the first project's URL on GitHub.com under the `dependencies` sub-property of `project` inside of a `cumulusci.yml` file.

You can see an example where I built out an "[org-agnostic utils](https://github.com/kkgthb/sf-devops-02-org-agnostic-utils)" CumulusCI project with a couple of Apex classes in it, then [included it as a dependency inside of a different CumulusCI project](https://github.com/kkgthb/sf-devops-04-that-one-feature-human-resources-wanted/blob/main/cumulusci.yml).

Salesforce's [David Reed](https://www.ktema.org/) says:

> "When you use a (GitHub repository without any releases on file) as a dependency, **CumulusCI will** see that there are no releases and **fall back to deploying** the **latest commit** on the main branch as unmanaged metadata."

For those of you who like to organize your Salesforce projects into "releases" and "dependencies," stay tuned -- hopefully, I'll find some time to start playing around with "minimum viable packaging & release" examples.

---

## Related experiments

* [This repo, but with Git tags](https://github.com/kkgthb/sf-cci-03-tag-but-no-sf-package) _(no Salesforce packaging yet)_
* [This repo, but with Git tags and GitHub releases](https://github.com/kkgthb/sf-cci-02-gh-release-exists-no-sf-package) _(no Salesforce packaging yet)_
* [This repo, but I also made a Salesforce package out of it](https://github.com/kkgthb/sf-cci-04-sf-package-without-gh-release) _(no Git tags or GitHub Releases)_

---

## Share your wins

[Let me know what you think](https://katiekodes.com/cci-minimum-viable-build/)

-[Katie Kodes](https://katiekodes.com/)