---
layout: post
title: Brew broke my things (again)
---
So I got bitten by homebrew, again. I wanted to update rabbitmq, so first update our local formulae cache:
```
> brew update
...
You have 127 outdated formulae and 3 outdated casks installed.
You can upgrade them with brew upgrade
```

Ok so far so good, I don't want to update 127 outdated formulae, but only this particular formula. So instead I ran this:

```
> brew upgrade rabbitmq
==> Upgrading 1 outdated package:
rabbitmq 3.9.14 -> 3.11.7
==> Fetching dependencies for rabbitmq: ca-certificates, openssl@1.1, unixodbc, jpeg-turbo, libpng, lz4, xz, libtiff, pcre2, wxwidgets and erlang
...
==> Summary
🍺  /usr/local/Cellar/rabbitmq/3.11.7: 1,412 files, 33.6MB
```

Yay! But u-oh what's this:
```
==> Running `brew cleanup rabbitmq`...
Disable this behaviour by setting HOMEBREW_NO_INSTALL_CLEANUP.
Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).
Removing: /usr/local/Cellar/rabbitmq/3.9.14... (1,391 files, 30.2MB)
xcrun: error: invalid active developer path (/Library/Developer/CommandLineTools), missing xcrun at: /Library/Developer/CommandLineTools/usr/bin/xcrun
Warning: The following dependents of upgraded formulae are outdated but will not
be upgraded because they are not bottled:
  mssql-tools
  msodbcsql17
==> Upgrading 53 dependents of upgraded formulae:
Disable this behaviour by setting HOMEBREW_NO_INSTALLED_DEPENDENTS_CHECK.
Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).
aom 3.3.0 -> 3.5.0_1, azure-cli 2.35.0 -> 2.44.1, borgbackup 1.2.0 -> 1.2.3, fbthrift 2022.03.21.00 -> 2023.01.16.00, folly 2022.03.21.00 -> 2023.01.16.00, fb303 2022.03.21.00 -> 2023.01.16.00, fish 3.3.1 -> 3.6.0, fizz 2022.03.21.00 -> 2023.01.16.00, freetype 2.12.0 -> 2.12.1, fontconfig 2.14.0 -> 2.14.1, gd 2.3.3_2 -> 2.3.3_4, gdk-pixbuf 2.42.8 -> 2.42.10, ghostscript 9.55.0 -> 10.0.0, git 2.35.1 -> 2.39.1, git-gui 2.35.1 -> 2.39.1, glib 2.72.0 -> 2.74.4, gnupg 2.3.4 -> 2.4.0, gnutls 3.7.4 -> 3.7.8_1, gobject-introspection 1.72.0 -> 1.74.0, gopass 1.14.0 -> 1.15.3, harfbuzz 4.2.0 -> 6.0.0_1, ipython 8.2.0 -> 8.8.0, jpeg-xl 0.6.1 -> 0.7.0_1, graphviz 3.0.0 -> 7.0.6, libavif 0.10.0 -> 0.11.1, libheif 1.12.0_1 -> 1.14.2, libraw 0.20.2_1 -> 0.21.1, little-cms2 2.13.1 -> 2.14, mongosh 1.3.1 -> 1.6.2, netpbm 10.86.32 -> 10.86.37, nghttp2 1.47.0 -> 1.51.0, node 17.8.0 -> 19.4.0_1, node@14 14.19.1 -> 14.21.2_1, openjpeg 2.4.0 -> 2.5.0, openssl@3 3.0.2 -> 3.0.7, p11-kit 0.24.1 -> 0.24.1_1, python@3.10 3.10.2 -> 3.10.9, pipenv 2022.4.8 -> 2022.12.19, python@3.8 3.8.13 -> 3.8.16, python@3.9 3.9.12 -> 3.9.16, imagemagick 7.1.0-29 -> 7.1.0-57, pango 1.50.6 -> 1.50.12, librsvg 2.52.8 -> 2.55.1, ruby 3.1.1 -> 3.2.0, rust 1.59.0 -> 1.66.1, tcl-tk 8.6.12_1 -> 8.6.13, shared-mime-info 2.1 -> 2.2, typescript 4.6.3 -> 4.9.4, unbound 1.15.0 -> 1.17.1, wangle 2022.03.21.00 -> 2023.01.16.00, watchman 2022.03.21.00 -> 2023.01.16.00, webp 1.2.2 -> 1.3.0, wget 1.21.3 -> 1.21.3_1
```

I didn't ask for this! However as this ran in a background window, by the time I noticed it was already too late. It does print the following though:
```
==> No broken dependents to reinstall!
```

Well it did update all the things, but at least it comforts me by telling me it didn't break anything. But u-oh, that appears to be a lie:

```
> yarn
dyld[51012]: Library not loaded: /usr/local/opt/icu4c/lib/libicui18n.70.dylib
  Referenced from: <1E8B97E7-7368-38ED-AF76-D71BC01206DE> /usr/local/Cellar/node@14/14.19.1/bin/node
  Reason: tried: '/usr/local/opt/icu4c/lib/libicui18n.70.dylib' (no such file), '/System/Volumes/Preboot/Cryptexes/OS/usr/local/opt/icu4c/lib/libicui18n.70.dylib' (no such file), '/usr/local/opt/icu4c/lib/libicui18n.70.dylib' (no such file), '/usr/local/lib/libicui18n.70.dylib' (no such file), '/usr/lib/libicui18n.70.dylib' (no such file, not in dyld cache), '/usr/local/Cellar/icu4c/72.1/lib/libicui18n.70.dylib' (no such file), '/System/Volumes/Preboot/Cryptexes/OS/usr/local/Cellar/icu4c/72.1/lib/libicui18n.70.dylib' (no such file), '/usr/local/Cellar/icu4c/72.1/lib/libicui18n.70.dylib' (no such file), '/usr/local/lib/libicui18n.70.dylib' (no such file), '/usr/lib/libicui18n.70.dylib' (no such file, not in dyld cache)
[1]    51012 abort      yarn
```

