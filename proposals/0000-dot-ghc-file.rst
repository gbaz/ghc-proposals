.. proposal-number:: Leave blank. This will be filled in when the proposal is
                     accepted.

.. trac-ticket:: Leave blank. This will eventually be filled with the Trac
                 ticket number which will track the progress of the
                 implementation of the feature.

.. implemented:: Leave blank. This will be filled in with the first GHC version which
                 implements the described feature.

.. highlight:: haskell

This proposal is `discussed at this pull request <https://github.com/ghc-proposals/ghc-proposals/pull/0>`_. **After creating the pull request, edit this file again, update the number in the link, and delete this bold sentence.**

.. contents::

Dot GHC File
==============
This proposal adds a new settings file for users, a ``.ghc`` file, intended as a complement to ``.ghci`` files. While ``.ghci`` files contain ghci commands, ``.ghc`` files will contain ghc flags.

Motivation
------------
Users of build tools such as cabal-install and stack can specify which flags are to be invoked for building packages in existing package config files such as cabal files, cabal.project files, or stack.yaml files. However, command-line users of the `ghc` executable itself have no such conveniences. This can lead to frustration when people want more or less verbosity by default (e.g. a recent discussion on if build output should include package paths). Additionally, the ability to specify default flags can be used when people are in a setting where they are working deep within examining some output and always want to display simplifications, force rebuilds, specify custom paths for extra lib or include dirs within their system, etc.

Proposed Change Specification
-----------------------------
This proposal adds ``.ghc`` files, which may be located and searched in the same way as ``.ghci`` files. Such files contain linebreak-delimited lists of ghc command line flags, and are executed in this order, if such files exist:

* ./.ghc
* appdata/ghc/ghc.conf
* On Unix: $HOME/.ghc/ghc.conf
* $HOME/.ghc

To prevent spooky action at a distance, files may contain *only that subset of flags that do not affect build output*. This is to say they may point ghc towards paths for resources, but may not specify optimization levels, code output targets, language extensions, etc.

Additionally, this adds a new ghc flag ``--no-dot-ghc-files`` that specifies that ghc is to disregard all such files. It is intended this flag be used for tools that provide their own flag management, such as cabal, stack, etc.

Effect and Interactions
-----------------------
As mentioned, the primary difficulty is making sure we do not cause adverse and confusing effects from stray ``.ghc`` files. The addition of the opt-out flag, which build tools are intended to use is one component of this. The other component is only allowing flags that do not directly affect the build product.

Costs and Drawbacks
-------------------
Since this rides on the coat-tails of an existing feature, it should be easy to implement, with the hardest part being maintaining a good list whitelist of allowable extensions. Since extensions are already somewhat categorized, and tracking which ones do and do not affect output is a good thing regardless, this seems not unduely high.

Alternatives
------------
Users can always type all flags at each invocation, set a command-line alias to ghc, or always go through other tools which manage flags. However, the possibility of per-directory files here allows a fair amout of flexibility. This is a quality-of-life feature for GHC power users, and as such may be considered not worth even the modest effort necessary to implement. However, it also poses the general social advantage that any and all elements of ghc cosmetic behavior considered good for power users but difficult for newer users can be relegated to options for `.ghc` files, and as such it can also be considered a road to "quality of life" for beginners.
