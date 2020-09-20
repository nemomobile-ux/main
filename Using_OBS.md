# Using Open Build Service

This document covers use of Open Build Service as provided at
https://build.opensuse.org . When compared to the service at
https://build.merproject.org, the differences are in provided source
services and base operating systems.


## Source services

Through the source services, the code is downloaded and prepared for
packaging. Target here is to get version information from the tag of
Git repository and prepare the source file to be compatible with the
source definition at RPM SPEC.

Full example `_service`, described below:

```XML
<services>
  <service name="obs_scm">
    <param name="url">https://github.com/rinigus/pkg-libpostal.git</param>
    <param name="scm">git</param>
    <param name="revision">devel</param>
    <param name="extract">rpm/*</param>
    <param name="filename">libpostal</param>
    <param name="versionformat">@PARENT_TAG@.@TAG_OFFSET@</param>
  </service>
  <service mode="buildtime" name="tar" />
  <service mode="buildtime" name="recompress">
    <param name="file">*.tar</param>
    <param name="compression">xz</param>
  </service>
  <service mode="buildtime" name="set_version" />
</services>
```

In this example, we use `obs_scm` to download sources and prepare
them. The used parameters are

* Type of source control is specified by `scm` parameter

* Branch, tag, commit ID go to `revision`

* It is possible to extract some files used for build using
  `extract`. You need to extract SPEC or provide it separately. In the
  example, full `rpm/` subfolder of the source is extracted. It is
  also possible to extract files one-by-one by specifying `extract`
  parameter several times. As when you have multiple SPEC files in the
  same source, you could select one in `extract`.

* `filename` specifies the base filename. By default, name of
  repository is used. Target is to have the same name as you have in
  RPM SPEC `Name`. In the example, repository is `pkg-libpostal` and
  this is different from the package name - `libpostal`. Hence,
  `filename` is set.

* to get the version set, use `versionformat` as specified:
  `@PARENT_TAG@.@TAG_OFFSET@`. This will form version string in the
  given format by the last specified tag and number of commits after
  that. So, if you have just tagged your repository with 1.0.0 and
  triggered the build, version of the package will be 1.0.0.0 with the
  last number being zero as an offset. There are more options
  available to format the version string, see documentation for them.

The remaining services are run during build. These services are
required to transform the format used by OBS to the one used in the
build. Thus, we have to get the source file using into the same format
as in RPM SPEC. In the example, RPM SPEC has
`Source0: %{name}-%{version}.tar.xz`. So, we use two services: `tar` and `recompress`.
For `recompress`, we have to specify

* the format using `compression` parameter

* which files to compress using `file` parameter, usually `*.tar`

Finally, use `set_version` service to parse and alter RPM SPEC by
setting `version` as determined by `obs_scm` service.


## Rebuilds

To trigger rebuild, it is sufficient to press the button "Trigger
services" on the package page.


## Dependencies

To check why packages are pulled as build dependencies, you could use

```
osc buildinfo -d Fedora_32 armv7l
```

in the package folder. In the example above, it was done for `Fedora_32` 
repository with `armv7l` architecture, same options as used for `osc build`.


## Useful links

To check available services and their documentation, run

```
osc api /service

```

Documentation

* [Source services](https://openbuildservice.org/help/manuals/obs-user-guide/cha.obs.source_service.html)

* [Best practicies](https://openbuildservice.org/help/manuals/obs-user-guide/cha.obs.best-practices.scm_integration.html)

* [Source for obs_scm](https://github.com/openSUSE/obs-service-tar_scm)

* [Tips](https://en.opensuse.org/openSUSE:Build_Service_Tips_and_Tricks)

* [Source service concept and list](https://en.opensuse.org/openSUSE:Build_Service_Concept_SourceService)

* [What is allowed to package](https://en.opensuse.org/openSUSE:Build_Service_application_blacklist)
