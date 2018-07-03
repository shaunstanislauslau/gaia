|logo|

|build-status| |go-report| |go-doc| |apache2| |chat|

gaia is an open source automation platform which makes it easy and fun
to build powerful pipelines in any programming language. Based on 
`HashiCorp's go-plugin`_ and `gRPC`_, gaia is efficient, fast, lightweight
and developer friendly. Gaia is currently alpha! `Do not use it for mission 
critical jobs yet!`_

Develop pipelines with the help of SDKs (currently only Go) and simply check-in your code into a git repository. Gaia automatically clones your code repository, compiles your code to a binary and executes it on-demand. All results are streamed back and formatted to a user-friendly graphical output.

Motivation
==========

.. begin-motivation

*Automation Engineer*, *DevOps*, *SRE*, *Cloud Engineer*, 
*Platform Engineer* - they all have one in common: 
The majority of tech people are not motivated to take up this work and they are hard to recruit.

One of the main reasons for this is the abstraction and poor execution of many automation tools. They come with their own configuration (`YAML`_ syntax) specification or limit the user to one specific programming language. Testing is nearly impossible because most automation tools lack the ability to mock services and subsystems. Even tiny things, for example parsing a JSON file, are sometimes really painful because external, outdated libraries were used and not included in the standard framework.

We believe it's time to remove all these abstractions and come back to our roots. Are you tired of writing endless lines of YAML-code? Are you sick of spending days forced to write in a language that does not suit you and is not fun at all? Do you enjoy programming in a language you like? Then gaia is for you.

How does it work?
=================

.. begin-architecture

Gaia is based on `HashiCorp's go-plugin`_. It's a plugin system that uses `gRPC`_ to communicate over HTTP2. HashiCorp developed this tool initially for `Packer`_ but it's now also heavily used by `Terraform`_, `Nomad`_, and `Vault`_.

Pipelines can be written in any programming language (gRPC support is a prerequisite) and can be compiled locally or simply over the build system. Gaia clones the git repository and automatically builds the included pipeline. 

After a pipeline has been started, all log output from the included jobs are returned back to gaia and displayed in a detailed overview with their final result status.

Gaia uses for storage `boltDB`_. This makes the installation step super easy. No external database is currently required.

Screenshots
===========

.. begin-screenshots


Getting Started
===============

.. begin-getting-started

Installation
------------

The installation of gaia is simple and often takes a few minutes.

Using docker
~~~~~~~~~~~~

The following command starts gaia as a daemon process and mounts all data to the current folder. Afterwards, gaia will be available on the host system on port 8080. Use the standard user **admin** and password **admin** as initial login. It is recommended to change the password afterwards.

.. code:: sh

    docker run -d -p 8080:8080 -v $PWD:/data gaiapipeline/gaia:latest

Manually
~~~~~~~~

It is possible to install gaia directly on the host system.
This can be achieved by downloading the binary from the `releases page`_.

gaia will automatically detect the folder of the binary and will place all data next to it. You can change the data directory with the startup parameter *--homepath* if you want.

Usage
-----

Go
~~~
Writing a pipeline is easy as importing a library, defining a function which will be the job to execute and serving the gRPC-Server via one command.

Here is an example:

.. code:: go

    package main

    import (
	    "log"

	    sdk "github.com/gaia-pipeline/gosdk"
    )

    // This is one job. Add more if you want.
    func DoSomethingAwesome() error {
	    log.Println("This output will be streamed back to gaia and will be displayed in the pipeline logs.")

	    // An error occured? Return it back so gaia knows that this job failed.
	    return nil
    }

    func main() {
	    jobs := sdk.Jobs{
		    sdk.Job{
			    Handler:     DoSomethingAwesome,
			    Title:       "DoSomethingAwesome",
			    Description: "This job does something awesome.",

			    // Increase the priority if this job should be executed later than other jobs.
			    Priority: 0,
		    },
	    }

	    // Serve
	    if err := sdk.Serve(jobs); err != nil {
		    panic(err)
	    }
    }

Like you can see, pipelines are defined by jobs. Usually, a function represents a job. You can define as many jobs in your pipeline as you want.

At the end, we define a jobs array that populates all jobs to gaia. We also add some information like a title, a description and the priority. 

The priority is really important and should always be used. If, for example, job A has a higher priority (decimal number) as job B, job A will be executed **after** job B. Priority defines therefore the order of execution. If two or more jobs have the same priority, those will be executed simultanously. You can compare it with the `Unix nice level`_.



.. _`HashiCorp's go-plugin`: https://github.com/hashicorp/go-plugin
.. _`gRPC`: https://grpc.io/
.. _`Do not use it for mission critical jobs yet!`: https://tenor.com/view/enter-at-your-own-risk-gif-8912210
.. _`YAML`: https://en.wikipedia.org/wiki/YAML
.. _`releases page`: https://github.com/gaia-pipeline/gaia/releases
.. _`Packer`: https://www.packer.io/
.. _`Terraform`: https://www.terraform.io/
.. _`Nomad`: https://www.nomadproject.io/
.. _`Vault`: https://www.vaultproject.io/
.. _`boltDB`: https://github.com/coreos/bbolt
.. _`Unix nice level`: https://en.wikipedia.org/wiki/Nice_(Unix)

.. |logo| image:: https://cdn.rawgit.com/michelvocks/ef3894f63c3bb004bca1a2fd5f7eb644/raw/c36d614db8afe229b466b38de1636a82ad809f64/gaia-logo-text.png
    :alt: gaia Logo
    :width: 650px

.. |build-status| image:: https://img.shields.io/circleci/project/github/gaia-pipeline/gaia/master.svg?label=circle&maxAge=43200
    :alt: Build Status
    :scale: 100%
    :target: https://circleci.com/gh/gaia-pipeline/gaia/tree/master

.. |go-report| image:: https://goreportcard.com/badge/github.com/gaia-pipeline/gaia
    :alt: Go Report Card
    :target: https://goreportcard.com/report/github.com/gaia-pipeline/gaia

.. |go-doc| image:: https://godoc.org/github.com/gaia-pipeline/gaia?status.svg
    :alt: GoDoc
    :target: https://godoc.org/github.com/gaia-pipeline/gaia

.. |apache2| image:: https://img.shields.io/badge/license-Apache-blue.svg
    :alt: Apache licensed
    :target: https://github.com/gaia-pipeline/gaia/blob/master/LICENSE

.. |chat| image:: https://img.shields.io/gitter/room/nwjs/nw.js.svg   
    :alt: Gitter
    :target: https://gitter.im/gaia-pipeline