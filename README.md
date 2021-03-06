# Tor.framework

[![Carthage Compatible](https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat)](https://github.com/Carthage/Carthage) 
[![Travis CI](https://img.shields.io/travis/iCepa/Tor.framework.svg)](https://travis-ci.org/iCepa/Tor.framework)

Tor.framework is the easiest way to embed Tor in your iOS application. The API is *not* stable yet, and subject to change.

Currently, the framework compiles in static versions of `tor`, `libevent`, `openssl`, and `liblzma`:

|          |         |
|:-------- | -------:|
| tor      | 0.4.0.5 |
| libevent | 2.1.8   |
| OpenSSL  | 1.1.0k  |
| liblzma  | 5.2.3   |

## Requirements

- iOS 8.0 or later
- Xcode 7.0 or later
- `autoconf`,  `automake`,  `libtool` and  `gettext` in your `PATH`

## Installation

Embedded frameworks require a minimum deployment target of iOS 8 or OS X Mavericks (10.9).

If you use `brew`, make sure to install `autoconf`,  `automake`,  `libtool` and  `gettext`:

```
brew install automake autoconf libtool gettext
```

### Initial Setup

```bash
git clone git@github.com:iCepa/Tor.framework

cd Tor.framework

git submodule init
git submodule update

carthage build --no-skip-current --platform iOS
```

### Carthage

To integrate Tor into your Xcode project using Carthage, specify it in your  `Cartfile`:

```ogdl
github "iCepa/Tor.framework" "master"
```

The above method will configure Carthage to fetch and compile Tor.framework from source. 
Alternatively, you may use the following to use binary-compiled versions of Tor.framework that 
correspond to releases in GitHub:

```ogdl
binary "https://icepa.github.io/Tor.framework/Tor.json" == 400.5.1
```

For available precompiled versions, see [docs/Tor.json](docs/Tor.json). Since Tor 0.3.5.2, 
the Tor.framework release version numbers follow the format "ABB.C.X" for tor version "0.A.B.C" 
and Tor.framework release X (for that version of Tor). Note that the "BB" slot is a two-digit number, 
with a leading zero, if necessary. "305.2.1" is the first release from tor 0.3.5.2.

#### Building a Carthage Binary archive

For maintainers/contributors of Tor.framework, a new precompiled release can be generated by 
doing the following:

Ensure that you have committed changes to the submodule trees for tor, libevent, openssl, and xz.

In `Tor/version.sh`, increment the `TOR_BUNDLE_SHORT_VERSION_STRING` version per the 
format described above. Change `TOR_BUNDLE_SHORT_VERSION_DATE` to the current date. 
Commit these changes.

Create a git tag for the version, and then 
[build + archive the framework](https://github.com/Carthage/Carthage/#archive-prebuilt-frameworks-into-one-zip-file):

```bash
carthage build --no-skip-current

carthage archive Tor
```
(This generates a `Tor.framework.zip` file in the repo.)

Then create a [release](https://github.com/iCepa/Tor.framework/releases) in GitHub which corresponds
to the tag, attach the generated `Tor.framework.zip` to the release.

Add a corresponding entry to [docs/Tor.json](docs/Tor.json), commit & push that so that it becomes 
available at https://icepa.github.io/Tor.framework/Tor.json

## Usage

Starting an instance of Tor involves using three classes: `TORThread`, `TORConfiguration` and `TORController`.

Here is an example of integrating Tor with `NSURLSession`:

```objc
TORConfiguration *configuration = [TORConfiguration new];
configuration.cookieAuthentication = @(YES);
configuration.dataDirectory = [NSURL URLWithString:NSTemporaryDirectory()];
configuration.controlSocket = [configuration.dataDirectory URLByAppendingPathComponent:@"control_port"];
configuration.arguments = @[@"--ignore-missing-torrc"];

TORThread *thread = [[TORThread alloc] initWithConfiguration:configuration];
[thread start];

NSURL *cookieURL = [configuration.dataDirectory URLByAppendingPathComponent:@"control_auth_cookie"];
NSData *cookie = [NSData dataWithContentsOfURL:cookieURL];
TORController *controller = [[TORController alloc] initWithSocketURL:configuration.controlSocket];
[controller authenticateWithData:cookie completion:^(BOOL success, NSError *error) {
    if (!success)
        return;

    [controller addObserverForCircuitEstablished:^(BOOL established) {
        if (!established)
            return;

        [controller getSessionConfiguration:^(NSURLSessionConfiguration *configuration) {
            NSURLSession *session = [NSURLSession sessionWithConfiguration:configuration];
            ...
        }];
    }];
}];
```

## Known Issues

- Nobody takes care having it working on MacOS, currently, so builds will probably break on that platform.

- Carthage warns about the xcconfigs dependency being seemingly unused.
  It isn't. It's only xcconfig files containing build settings, so nothing actually ends up in the build
  prodcut. Unfortunately Carthage can't be configured to not throw this warning.

## License

Tor.framework is available under the MIT license. See the 
[`LICENSE`](https://github.com/iCepa/Tor.framework/blob/master/LICENSE) file for more info.
