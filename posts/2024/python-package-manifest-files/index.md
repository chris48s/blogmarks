<!--
.. title: An analysis of python package manifest files
.. slug: python-package-manifest-files
.. date: 2024-02-17 00:00:00
.. tags: python,packaging
.. category: 
.. link: 
.. description: To be a proper python dev these days you have to either build your own python package manager in rust, or write a blog post about the state of python packaging. Turns out I am crap at rust.
.. type: text
-->

Python packaging is messy and fragmented. Lots of people have been writing about it recently and there have been some great articles that have attracted a lot of attention. For example, I've particularly enjoyed:

- [An unbiased evaluation of environment management and packaging tools](https://alpopkes.com/posts/python/packaging_tools/) by Anna-Lena Popkes
- [How to improve Python packaging, or why fourteen tools are at least twelve too many](https://chriswarrick.com/blog/2023/01/15/how-to-improve-python-packaging/) by Chris Warrick and
- [Thoughts on the Python packaging ecosystem](https://pradyunsg.me/blog/2023/01/21/thoughts-on-python-packaging/) by Pradyun Gedam

Gregory Szorc also captured the frustrating experience many developers face trying to navigate the modern python packaging landscape in [My User Experience Porting Off setup.py](https://gregoryszorc.com/blog/2023/10/30/my-user-experience-porting-off-setup.py/).

It is a topic I also spend a lot of time thinking about, but I decided to take a look at the topic from a slightly different angle. Instead of lamenting the proliferation of different tools, or attempting to round them all up and compare them, I decided to ask: What are package authors actually doing out there in the wild, and how is the community responding to this change and fragmentation?

So I conducted a bit of research. I looked at a sample of 31,474 public GitHub repos associated with one or more python packages on PyPI and analysed the manifests to find out a bit more about how people are actually specifying their package metadata and building their packages. For the purposes of this research, I'm focussing on packages. You could probably ask and answer some similarly interesting questions about applications, but I haven't done it here. There's a bit more information about how and why I arrived at this sample of ~30k GitHub repos in the [methodology notes](#methodology-notes), but I'm not going to bury the lead. Lets just jump straight into the good stuff.


## Manifest files

I looked for the presence of 3 files: pyproject.toml, setup.py and setup.cfg. Most of the repos I looked at contained more than one.

<table>
  <tr>
    <th>File</th>
    <th>Count</th>
    <th>Percent</th>
  </tr>
  <tr>
    <td>setup.py</td>
    <td class="right-align">20,684</td>
    <td class="percent" style="background-size: 66% 100%">66%</td>
  </tr>
  <tr>
    <td>pyproject.toml</td>
    <td class="right-align">17,245</td>
    <td class="percent" style="background-size: 55% 100%">55%</td>
  </tr>
  <tr>
    <td>setup.cfg</td>
    <td class="right-align">10,406</td>
    <td class="percent" style="background-size: 33% 100%">33%</td>
  </tr>
  <tr>
    <td><b>Total</b></td>
    <td class="right-align"><b>31,474</b></td>
    <td><b>-</b></td>
  </tr>
</table>


## Pyproject.toml

One of the big pushes in python is for adoption of pyproject.toml. So how is that going out there in the real world?

First of all, it is worth reviewing some of the ways pyproject.toml is or can be used.

- [PEP 517](https://peps.python.org/pep-0517/) Defines a way to declare a package build backend in pyproject.toml.
- [PEP 621](https://peps.python.org/pep-0621/) Defines a way to define the package metadata in pyproject.toml.
- [PEP 518](https://peps.python.org/pep-0518/) Defines a way to declare package build requirements in pyproject.toml and a way for python tools (which may or may not be related to packaging) to store configuration in the `tool.*` namespace. Many python tools like pytest, black, mypy, etc allow their configuration to be stored in pyproject.toml using the `tool.*` namespace.
- In particular, poetry allows package metadata to be specified in pyproject.toml in a `tool.poetry` declaration, but predates and does not conform to PEP 621. I'm going to consider poetry separately.

A point to note here is that these can be combined in various ways. For example, it is possible to declare a build backend in pyproject.toml following PEP 517 and also declare PEP 621 package metadata. However using setuptools it is also possible to declare a build backend in pyproject.toml but specify the rest of the package metadata in setup.py or setup.cfg. Some repos only use pyproject.toml for storing linter configuration and everything to do with packaging is stored in setup.py or setup.cfg. Some repos specify package metadata in pyproject.toml (either following PEP 621 or using poetry), but don't declare a build system. One does not necessarily imply another. I found examples of pretty much every combination. This makes it difficult to conduct a completely coherent analysis or arrive at universally valid assumptions.

In the sample of repos I looked at 17,245 (55%) contained a pyproject.toml file. 15,754 (91%) of those declare a build backend, requirements, and/or package metadata. 1,497 (9%) did not contain any of those things. Presumably in basically all of those cases, pyproject.toml is being used exclusively as a configuration file for dev tooling.

<table>
  <tr>
    <th>Feature</th>
    <th>Count</th>
    <th>Percent</th>
  </tr>
  <tr>
    <td>Has build requirements</td>
    <td class="right-align">15,427</td>
    <td class="percent" style="background-size: 89% 100%">89%</td>
  </tr>
  <tr>
    <td>Has build backend</td>
    <td class="right-align">14,328</td>
    <td class="percent" style="background-size: 83% 100%">83%</td>
  </tr>
  <tr>
    <td>Has PEP 621 metadata</td>
    <td class="right-align">6,563</td>
    <td class="percent" style="background-size: 38% 100%">38%</td>
  </tr>
  <tr>
    <td>Has Poetry metadata</td>
    <td class="right-align">4,890</td>
    <td class="percent" style="background-size: 28% 100%">28%</td>
  </tr>
  <tr>
    <td>Has no packaging metadata</td>
    <td class="right-align">1,497</td>
    <td class="percent" style="background-size: 9% 100%">9%</td>
  </tr>
  <tr>
    <td><b>Total</b></td>
    <td class="right-align"><b>17,245</b></td>
    <td><b>-</b></td>
  </tr>
</table>

There are a few interesting results here. The first is that most repos containing a pyproject.toml declare either a build backend and/or requirements. I was actually surprised that more files declare build requirements than a build backend. I expected repos declaring build requirements would basically be a subset of those declaring a build backend. Turns out the inverse is true.

Many repos are declaring package metadata in pyproject.toml using either PEP 621 or Poetry format, but adoption of pyproject.toml for this purpose is less common.

My hunch is that a lot of the repos which are only specifying build backend/requirements may have adopted pyproject.toml primarily as a configuration format (as opposed to a package manifest format) and then added a minimal `build-system` declaration for compatibility purposes. However that is just my conjecture.


## Setup.py and setup.cfg

The oldest way to specify package metadata is using setup.py. This has served the community well for many years, but the package metadata is mixed with executable python code. The python community's first attempt at a declarative manifest format was setup.cfg. This was a format specific to setuptools rather than a standard and the setuptools project plans to [eventually deprecate](https://github.com/pypa/setuptools/issues/3214) setup.cfg. One of the big pushes in python is for moving away from setup.py and setup.cfg to specify package metadata, and towards pyproject.toml. So how is that going out there in the real world?

Of the repos I looked at, 20,684 (66%) contained a setup.py and 10,406 (33%) contained a setup.cfg file. Many contained both. As with pyproject.toml, presence or absence of the file in a repo doesn't necessarily tell us the full story. Some repos that are primarily using pyproject.toml also have a stub setup.py that just contains

```py
import setuptools
setuptools.setup()
```

for backwards compatibility reasons. This may be needed, for example for compatibility with tools that don't support [PEP 660](https://peps.python.org/pep-0660/) editable installs.

As with pyproject.toml, many python based tools like isort and flake8 allow their configuration to be stored in setup.cfg so some repos contain a setup.cfg but aren't using to store any information related to packaging - it is just there to store linter configuration. Again, basically every combination of scenarios exists in the sample of repos I looked at.

I haven't attempted to parse the setup.py and setup.cfg files. I am perhaps missing a bit of nuance here, but I have made some assumptions:

- A repo which declares poetry or PEP 621 package metadata in pyproject.toml is using pyproject.toml as the package manifest.
- A repo that has a setup.py but not a setup.cfg and either doesn't have a pyproject.toml at all or has a pyproject.toml which does not contain poetry or PEP 621 package metadata is using setup.py as the package manifest.
- A repo that has a setup.py and setup.cfg and either doesn't have a pyproject.toml at all or has a pyproject.toml which does not contain poetry or PEP 621 package metadata is using setup.py or setup.cfg as the package manifest.
- There were also just over 1,000 repos doing some other combination of things. A lot of these were a pyproject.toml declaring a build system or build requirements only, with metadata in setup.cfg. I didn't attempt to break them down any further.

<table>
  <tr>
    <th>Manifest type</th>
    <th>Count</th>
    <th>Percent</th>
  </tr>
  <tr>
    <td>pyproject.toml with metadata</td>
    <td class="right-align">11,349</td>
    <td class="percent" style="background-size: 36% 100%">36%</td>
  </tr>
  <tr>
    <td>setup.py only</td>
    <td class="right-align">10,695</td>
    <td class="percent" style="background-size: 34% 100%">34%</td>
  </tr>
  <tr>
    <td>setup.py and setup.cfg</td>
    <td class="right-align">8,235</td>
    <td class="percent" style="background-size: 26% 100%">26%</td>
  </tr>
  <tr>
    <td>Other</td>
    <td class="right-align">1,195</td>
    <td class="percent" style="background-size: 4% 100%">4%</td>
  </tr>
  <tr>
    <td><b>Total</b></td>
    <td class="right-align"><b>31,474</b></td>
    <td class="percent" style="background-size: 100% 100%"><b>100%</b></td>
  </tr>
</table>

18,930 (63%) of the repos I looked at are sticking with setup.py and/or setup.cfg as the package manifest.

Using only a setup.py is still a very popular method of packaging at 34%. This is nearly equal with storing package metadata in pyproject.toml at 36%, despite efforts to transition the community away from executable package manifests and towards declarative manifest formats.


## Build Backends

14,328 of the repos I looked at are using a pyproject.toml that declares a PEP-517 build backend. So next I dug into that. Which build backends are these repos using?

<table>
  <tr>
    <th>Build backend</th>
    <th>Count</th>
    <th>Percent</th>
  </tr>
  <tr>
    <td>Setuptools</td>
    <td class="right-align">6,732</td>
    <td class="percent" style="background-size: 47% 100%">47%</td>
  </tr>
  <tr>
    <td>Poetry</td>
    <td class="right-align">4,671</td>
    <td class="percent" style="background-size: 33% 100%">33%</td>
  </tr>
  <tr>
    <td>Hatch</td>
    <td class="right-align">1,592</td>
    <td class="percent" style="background-size: 11% 100%">11%</td>
  </tr>
  <tr>
    <td>Flit</td>
    <td class="right-align">687</td>
    <td class="percent" style="background-size: 5% 100%">5%</td>
  </tr>
  <tr>
    <td>Other</td>
    <td class="right-align">223</td>
    <td class="percent" style="background-size: 2% 100%">2%</td>
  </tr>
  <tr>
    <td>Pdm</td>
    <td class="right-align">215</td>
    <td class="percent" style="background-size: 2% 100%">2%</td>
  </tr>
  <tr>
    <td>Maturin</td>
    <td class="right-align">208</td>
    <td class="percent" style="background-size: 1% 100%">1%</td>
  </tr>
  <tr>
    <td><b>Total</b></td>
    <td class="right-align"><b>14,328</b></td>
    <td class="percent" style="background-size: 100% 100%"><b>100%</b></td>
  </tr>
</table>

There are more interesting findings here:

- Among repos using pyproject.toml, setuptools is the by far the most commonly declared build backend, accounting for nearly half the repos I looked at.
- New shiny tools like poetry, hatch and flit have some adoption, but account for a much smaller share of the ecosystem.
- By far the most widely used of these more modern packaging tools is poetry, accounting for 33% of the repos I looked at declaring a build backend in pyproject.toml.


## Setuptools

Finally, I wanted to look at those repos using setuptools and pyproject.toml. Broadly, these are going to divide into 2 camps:

- Those specifying package metadata in pyproject.toml, following PEP-621
- Those specifying a build backend only in pyproject.toml, following PEP-517, but storing the package metadata in setup.py or setup.cfg.

<table>
  <tr>
    <th>Metadata location</th>
    <th>Count</th>
    <th>Percent</th>
  </tr>
  <tr>
    <td>Outside pyproject.toml</td>
    <td class="right-align">3,615</td>
    <td class="percent" style="background-size: 54% 100%">54%</td>
  </tr>
  <tr>
    <td>Inside pyproject.toml (PEP-621)</td>
    <td class="right-align">3,117</td>
    <td class="percent" style="background-size: 46% 100%">46%</td>
  </tr>
  <tr>
    <td><b>Total</b></td>
    <td class="right-align"><b>6,732</b></td>
    <td class="percent" style="background-size: 100% 100%"><b>100%</b></td>
  </tr>
</table>

Among repos using setuptools and pyproject.toml, only a minority have adopted PEP-621 for declaring package metadata. In the sample of repos I looked at which declare setuptools as a build backend in pyproject.toml, the most popular approach (albeit by a small margin) is to declare only the build backend details in pyproject.toml and store the package metadata elsewhere.


## Conclusions

Based on the analysis I've done here, it seems reasonable to say that adoption of pyproject.toml has been slow, particularly as a package manifest format. Most of the repos I looked at are only or primarily using setup.py and/or setup.cfg. Modern packaging tools are generating blog posts, debate, and mindshare. Out there in the real world we are seeing limited adoption in comparison to more traditional approaches. While a blog post about setuptools is less likely to hit the front page of hackernews, setuptools is the real workhorse when it comes to getting packages shipped.

As noted at the start of this article, python packaging is a confusing and fragmented space at the moment. There are a lot of ways to skin this cat. It seems reasonable to infer that as a response to this, many developers are choosing to stick with an existing working solution, rather than make sense of the chaos. Who can blame them?

The python community often moves slowly in response to change. For example the migration from python 2 to 3 dragged on for about a decade, but in that case the direction of travel was at least clear. There was a single linear path. When it comes to modernizing the packaging space, progress is also hindered by the fact that for some projects there are many possible directions of travel. For some projects, there are still zero. Perhaps this is a journey that will take even longer to shake out.


<h2 id="methodology-notes">Methodology notes</h2>

This research was based on a convenience sample. I looked at a selection of repos that made it quick and easy to harvest data, rather than the most robust sample or a complete census of PyPI.

As a starting point, I used the 2023-10-22 [Ecosyste.ms Open Data](https://packages.ecosyste.ms/open-data) Release (which is licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) ). This was an easy place to get a bulk list of python packages with GitHub repos attached. I then applied a few filters.

First I excluded any packages which didn't have one or more releases published inside 2023. I'm really looking into modern packaging practices, so packages without a recent release are less useful to consider here.

Then I excluded any packages that had less than 100 downloads in the last month. There is a lot of junk on PyPI. This is a low bar for popularity, but I wanted to apply some kind of measure of "someone is actually using this code for something". Applying even this modest filter excluded a surprisingly large number of packages.

Then finally, I looked only at packages which had a GitHub repo attached to them in the Ecosyste.ms data. This was mainly about making it easy to fetch data in bulk. This means I excluded repos hosted on GitLab, BitBucket, CodeBerg, etc from this analysis. I also did not attempt to look at packages that had no `repository_url` attached in the data. As such, the sample contains some blind spots.

After de-duplicating, this gave me 35,732 GitHub repository URLs.

I then used the GitHub GraphQL API to attempt to fetch a setup.py, setup.cfg and pyproject.toml if they existed in the repo root. After excluding any repos that were private, did not exist at all, or repos that didn't contain any of those files in the root, I was left with the 31,474 repos that formed the basis of this analysis. Another obvious blind spot here is repos that host a package in a subdirectory instead of the repo root. Those will have been excluded too.

Finally, I grabbed whatever files were at the `HEAD` of the default branch in GitHub. I didn't attempt to find a latest release, or the release that would have been current at the time of the ecosyste.ms open data release. I don't think this makes a huge difference, but it is worth noting.


## Future work

This has been an interesting process, but it only represents a snapshot in a landscape that is shifting over time. I'd like to repeat this analysis again in future to see how the things have changed. It's been a blast. Let's do it again some time.
