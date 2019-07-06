Atom CI
=======

This is a script for setting up continuous integration with an Atom project.
It's fundamentally the same as [`atom/ci`][], with the following differences:

1.	__GitHub's [APIs][] are consulted for release downloads.__  
	This is a tad bit slower than downloading from [`atom.io`][],
	but it means sudden changes to their infrastructure won't break your build.

2.	__Arbitrary release channels are unsupported.__  
	Only `stable` and `beta` releases of Atom can be tested against.

3.	__Only [TravisCI][] is supported.__

4.	__`lint` or `test` scripts defined in `package.json` are used, if possible.__  
	If your package manifest defines a `lint` or `test` script, the CI script will
	call those instead. For example:
	~~~json
	{
		"scripts": {
			"lint": "npx eslint --ext mjs,js ./lib/ ./tools",
			"test": "atom -t ./specs"
		}
	}
	~~~
	If you don't specify a script, the usual defaults assumed by [`atom/ci`][] are
	attempted instead:
	~~~shell
	# Linting
	DIRS="./lib ./src ./spec ./test"
	npx coffeelint $DIRS
	npx eslint $DIRS
	npx tslint $DIRS

	# Testing
	DIRS="./spec ./specs ./test ./tests"
	atom --test $DIRS
	~~~
	Note that only linters listed in `devDependencies` will be run, and missing
	directories will be skipped. However, at least one test directory *must* be
	included, or else the build will fail.


To-do list
----------
*	[ ] **Support an `${ATOM_RELEASE}` environment variable to test specific releases**  

*	[ ] **Support the `atom-mocha` executable, once it can be run globally**  

*	[ ] **Learn PowerShell and write a version of this for [AppVeyor][]**  
	Not a huge priority at the moment, as the AppVeyor integration provided
	by [`atom/ci`][] is currently working fine.


Background
--------------------------------------------------------------------------------
On July 6th 2019 AEST, Atom's CI script suddenly broke. The culprit was botched
handling of a [`curl(1)`](https://curl.haxx.se/docs/manpage.html) request which
included an `Accept` header:

~~~console
$ curl -L "https://atom.io/download/mac?channel=${ATOM_CHANNEL}" \
	-H 'Accept: application/octet-stream' \
	-o "atom.zip"
######################################################################## 100.0%
curl: (22) The requested URL returned error: 406 Not Acceptable
~~~

Following the URL simply lead to
Atom's [releases page](`https://github.com/atom/atom/releases/latest`) on GitHub.
I was unsure what the link usually pointed to, but having this break the builds
of each of my projects was certainly *not* the intended outcome.

Since I'm blocked from the @atom org on GitHub, I was unable to report this or
submit a pull-request. So, as usual, I took things into my own hands.



<!-- Referenced links -->
[APIs]: https://developer.github.com/v3/repos/releases/
[`atom/ci`]: https://github.com/atom/ci
[`atom.io`]: https://atom.io/
[TravisCI]: https://travis-ci.com/
[AppVeyor]: https://appveyor.com/