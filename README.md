# BetaBuilder, a gem for managing iOS ad-hoc builds

BetaBuilder is a simple collection of Rake tasks and utilities for managing and publishing Adhoc builds of your iOS apps. 

If you're looking for the OSX BetaBuilder app -- to which this gem owes most of the credit -- you can find it [here on Github](http://github.com/HunterHillegas/iOS-BetaBuilder).

**Note: As of release 0.7, support for Xcode 3 is deprecated. Xcode 4 has been out for a year, has got much more stable as of 4.1 on Lion (or 4.2 if you've been using the betas) and it's time to move on. Generating Xcode 4 friendly archives and builds in this release still needs a bit of configuration but will become much smoother in 0.8 once Xcode 3 support is removed.**

## Motivation

The problem with using a GUI app to create the beta packages is that it is yet another manual step in the process of producing an ad-hoc build for your beta testers. It simplifies some steps but it still requires running Build and Archive in Xcode, saving the resulting build as an IPA package, running the Beta Builder app, locating the IPA, filling in the rest of the fields and generating the deployment files. Then you need to upload those files somewhere.

As a Ruby developer, I use Rake in most of my projects to run repetitive, often build or test-related tasks and it's equally as useful for non-Ruby projects as it is for Ruby ones.

This simple task library allows you to configure once and then build, package and distribute your ad-hoc releases with a single command.

## Usage

To get started, if you don't already have a Rakefile in the root of your project, create one. If you aren't familiar with Rake, it might be worth [going over some of the basics](http://rake.rubyforge.org/) but it's fairly straightforward.

You can install the BetaBuilder gem from your terminal (OSX 10.6 ships with a perfectly useful Ruby installation):

    $ gem install betabuilder

At the top of your Rakefile, you'll need to require `rubygems` and the `betabuilder` gem (obviously).

    require 'rubygems'
    require 'betabuilder'
    
Because BetaBuilder is a Rake task library, you do not need to define any tasks yourself. You simply need to configure BetaBuilder with some basic information about your project and it will generate the tasks for you. A sample configuration might look something like this:

    BetaBuilder::Tasks.new do |config|
      # your Xcode target name
      config.target = "MyGreatApp"

      # the Xcode configuration profile
      config.configuration = "Adhoc" 
    end
    
Now, if you run `rake -T` in Terminal.app in the root of your project, the available tasks will be printed with a brief description of each one:

    rake beta:build     # Build the beta release of the app
    rake beta:package   # Package the beta release as an IPA file
    
If you use a custom Xcode build directory, rather than the default `${SRCROOT}/build` location, you can configure that too:

    BetaBuilder::Tasks.new do |config|
      ...
      config.build_dir = "/path/to/custom/build/dir"
    end

To deploy your beta to your testers, some additional configuration is needed (see the next section).

Most of the time, you'll not need to run the `beta:build` task directly; it will be run automatically as a dependency of `beta:package`. Upon running this task, your ad-hoc build will be packaged into an IPA file and will be saved in `${PROJECT_ROOT}/pkg/dist`, along with a HTML index file and the manifest file needed for over-the-air installation.

If you are not using the automatic deployment task, you will need to upload the contents of the pkg/dist directory to your server.

To use a namespace other than "beta" for the generated tasks, simply pass in your chosen namespace to BetaBuilder::Tasks.new:

    BetaBuilder::Tasks.new(:my_custom_namespace) do |config|
    end
    
This lets you set up different sets of BetaBuilder tasks for different configurations in the same Rakefile (e.g. a production and staging build).

## Configuration
A full list of configuration options and their details

`configuration` - (String) The Xcode Configuration to use (Defined on the Info tab of the Project)

`build_dir` - (File Path) The directory the build output will be. (`:derived` for Xcode 4 for versions < 0.8)

`auto_archive` - (true/**false**) Automatically archive when packaging

`archive_path` - (File Path) Path to the Archives

`xcodebuild_path` - (File Path) Path to xcodebuild, if its non-standard

`xcodeargs` - (Arguments) Arguments passed to `xcodebuild` when it runs

`project_file_path` - (File Path) Path to the `.xcodeproj` file

`workspace_path` - (File Path) Path to the `.xcworkspace` file

`ipa_destination_path` - (File Path) Path to output Packaged IPA to

`scheme` - (String) What Scheme to use when building

`app_name` - (String) The name of the app being built (should match the file name, less the `.app` extension)

`arch` - (String) The architecture to build for, if different from project settings

`xcode4_archive_mode` - (true/**false**) Use Xcode4's Archive mode

`skip_clean` - (true/**false**) Skip the clean step when building

`verbose` - (true/**false**) Increased output for debugging

`dry_run` - (true/**false**) Don't upload to Distribution Strategy, just act like it

`set_version_number_git` - (true/**false**) Attempts to set a version number using Git, see below

`set_version_number_svn` - (true/**false**) Attempts to set a version number using SVN, see below

### Configuration (Testflight)
Testflight presents its own set of options that can be configured

`api_token` - (String) Can be your API key, but its recommended to use an `ENV[""]` variable

`team_token` - (String) Your Team's Testflight API token

`distribution_lists` - (Array) A Ruby array (`[1,2]` or `%w{1 2}`) of distribution list names for Testflight

`notify` - (true/**false**) Notify the distribution list of this build

`ask_to_notify` - (true/**false**) Asks during the upload if the distribution list should be notified of this build, uses ‘notify’ as default value.

`replace` - (true/**false**) Replace if an existing build exists with the same ID and version

### Configuration (Web)
Pushing to a web server has the following options.

SSH keys will simplify authentication and make this process seamless

`remote_host` - (String) Hostname for the server the build will be pushed to

`remote_port` - (String) Port Number to use for SCP/SFTP

`remote_installation_path` - (String) Remote Path

### Configuration (RunTime)
Certain configuration options are availabe at the command line, so that you can temporarily set them for a single run without modifying your configuration.

Pass any of these in as environment variables:

`DRY` - (true/**false**) Enable Dry Run
`VERBOSE` - (true/**false**) Turn on all output; lets you see the clean, build, and signing output
`SKIPCLEAN` - (true/**false**) Skips the clean step and goes right to Build.

####Examples

`rake staging:deploy DRY=true`

`rake staging:redeploy VERBOSE=true SKIPCLEAN=true`

## Xcode 4 support

Betabuilder works with Xcode 4, but you may need to tweak your task configuration slightly. The most important change you will need to make is the build directory location, unless you have configured Xcode 4 to use the "build" directory relative to your project, as in Xcode 3.

If you are using the Xcode derived data directory for your builds, then you will need to specify this. Betabuilder will then scan your build log to determine the path to the automatically generated build directory that Xcode is using for your project.

    config.build_dir = :derived
    
This will become the default in 0.8.

If you wish to generate archives for your Xcode 4 project, you will need to enable this. This will become the default in future once Xcode 3 support is dropped (deprecated in 0.7):

    config.xcode4_archive_mode = true
    
If you are working with an Xcode 4 workspace instead of a project file, you will need to configure this too:

    config.workspace_path = "MyWorkspace.xcworkspace"
    config.scheme         = "My App Scheme"
    config.app_name       = "MyApp"
    set_version_number
If you are using a workspace, then you must specify the scheme. You can still specify the build configuration (e.g. Release).

## Automatic deployment with deployment strategies

BetaBuilder allows you to deploy your built package using it's extensible deployment strategy system; the gem currently comes with support for simple web-based deployment or uploading to [TestFlightApp.com](http://www.testflightapp.com). Eventually, you will be able to write your own custom deployment strategies if neither of these are suitable for your needs.

## Setting version numbers automatically

You can use betabuilder to set a build number using Git's `describe` system or Subversion's svnversion.  In order for Git to work, at least one `tag` must exist somewhere in the git hierarchy for the branch being built from. 

Also, you are required to set your `CFBundleVersion` to `${VERSION_LONG}` inside your `Info.plist`.  To accomodate manual builds, add a `VERSION_LONG` Build Setting to your app's Project, and treat it as you normally would your `Info.plist` version number.

Once a tag is created (if using Git) and your App is configured, simply add this to your BetaBuilder config and it will use Git to generate the 

	config.set_version_number_svn = true
	
OR

	config.set_version_number_git = true

It will generate a version number like: `build-3729` or `1.0-15-g6b3c1bb`. 

Git Tag Details: 

* *1.0* is the most recent tag
* *15* is the number of commits since the tag was generated
* *g6b3c1bb* is the beginning of the hash of the last commit.

### Deploying your app with TestFlight

By far the easiest way to get your beta release into the hands of your testers is using the excellent [TestFlight service](http://testflightapp.com/), although at the time of writing it is still in private beta. You can use TestFlight to manage your beta testers and notify them of new releases instantly.

TestFlight provides an upload API and betabuilder uses that to provide a `:testflight` upload strategy. This strategy requires two pieces of information: your TestFlight API token and your team token:

    config.deploy_using(:testflight) do |tf|
      tf.api_token  = "YOUR_API_TOKEN"
      tf.team_token = "YOUR_TEAM_TOKEN"
    end
    
Now, instead of using the `beta:package` task, you can run the `beta:deploy` task instead. This task will run the package task as a dependency and upload the generated IPA file to TestFlight. 

You will be prompted to enter the release notes for the build; TestFlight requires these to inform your testers of what has changed in this build. Alternatively, if you have a way of generating the release notes automatically (for instance, using a CHANGELOG file or a git log command), you can specify a block that will be called at runtime - you can do whatever you want in this block, as long as you return a string which will be used as the release notes, e.g.

    config.deploy_using(:testflight) do |tf|
      ...
      tf.generate_release_notes do
        # return release notes here
      end
    end
    
Finally, you can also specify an array of distribution lists that you want to allow access to the build:

    config.deploy_using(:testflight) do |tf|
      ...
      tf.distribution_lists = %w{Testers Internal}
    end

### Deploying to your own server

BetaBuilder also comes with a rather rudimentary web-based deployment task that uses SCP, so you will need SSH access to your server and appropriate permissions to use it. This works in the same way as the original iOS-BetaBuilder GUI app by generating a HTML template and manifest file that can be uploaded to a directly on your server. It includes links to install the app automatically on the device or download the IPA file.

You will to configure betabuilder to use the `web` deployment strategy with some additional configuration:

    config.deploy_using(:web) do |web|
      web.deploy_to = "http://beta.myserver.co.uk/myapp"
      web.remote_host = "myserver.com"
      web.remote_directory = "/remote/path/to/deployment/directory"
    end
    
The `deploy_to` setting specifies the URL that your app will be published to. The `remote_host` setting is the SSH host that will be used to copy the files to your server using SCP. Finally, the `remote_directory` setting is the path to the location to your server that files will be uploaded to. You will need to configure any virtual hosts on your server to make this work.

## License

This code is licensed under the MIT license.

Copyright (c) 2010 Luke Redpath

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
