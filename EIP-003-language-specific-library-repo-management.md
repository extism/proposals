#  Language-specific library repo management

## Purpose

Determine the near-term approach to repo management, specifically for the language SDKs / wrapper libraries over the core runtime.

## Solution

For now, it seems more beneficial to the project to contain all runtime and libraries within a single monolithic repo.

In the future, a "repo-per-language" approach might be more useful, so that contributors can more precisely find the code they're interested in,
or for deployment / packaging & distribution simplification. 

## Considerations

#### Maintainability

Having everything all in one place makes it a bit easier to track changes, build code, test, & work locally. CI is also simplified since there are
no cross-repo dependencies to worry about downloading.

#### End-user access

It would be simpler to point a end-user (e.g. a Go developer, importing Extism into their app) to the "Go-specific" SDK, and have all the pertinent 
documentation and examples on the main `README.md` of that repo. However, we should be able to distribute the language SDKs to respective package 
managers and point to the corresponding `README.md` and docs for that language.

#### Contributor complexity

While any can easily be linked to a language specific library (e.g. they only really care about using Extism in Go), there are similar issues
as CI, where cross-repo dependencies can be hard to managage. In a single repo, one `git clone ...` is enough to start working with the code.

#### Awareness & Discoverability

As the project is ony as useful if people know about it, we should also try to aggregate attention to a single place. Help people find the main 
project, and then can be linked to package managers, docs, examples of specific language SDK usage. Having multiple repos now would dilute the
focus and spread it across the individual SDKs. 

The counter-argument is that individual language communities might be more inclined to spread awareness of a repo that is specifically tailored to
the language of their community. 
