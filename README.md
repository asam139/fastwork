# Installation

Make sure you have the latest version of the Xcode command line tools installed:

```
xcode-select --install
```

Install _fastlane_ using
```
[sudo] gem install fastlane -NV
```
or alternatively using `brew install fastlane`

Config bundler to support the fastlane plugins adding:
```
plugins_path = File.join(File.dirname(__FILE__), 'fastlane', 'Pluginfile')
eval_gemfile(plugins_path) if File.exist?(plugins_path)
```

# Available Actions
## iOS
### ios lint
```
fastlane ios lint
```
Validate the project is ready for releasing
### ios patch
```
fastlane ios patch
```
Release a new version with a `patch` bump_type
### ios minor
```
fastlane ios minor
```
Release a new version with a `minor` bump_type
### ios major
```
fastlane ios major
```
Release a new version with a `major` bump_type
### ios test
```
fastlane ios test
```

# Environment

It places a file with the name `.env.fastlane` in the root folder to define all environment variables. The available variables can be found in the `.env.template`.

----

More information about fastlane can be found on [fastlane.tools](https://fastlane.tools).
The documentation of fastlane can be found on [docs.fastlane.tools](https://docs.fastlane.tools).
