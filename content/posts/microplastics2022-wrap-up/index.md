+++ 
draft = false
date = 2022-11-15T11:32:00+00:00
title = "Microplastics2022 conference wrap-up"
description = "A few scattered reflections on the Microplastics2022 conference in Monte Verita"
tags = ["microplastics"]
images = ["posts/microplastics2022-wrap-up/group-photo.jpeg"]
+++

Sitting on the TGV from Zurich to Paris, returning from an exciting and educational week spent at the [Microplastics2022 conference](https://microplastics2022.ch/) in Monte Verità in Switzerland, I thought I should jot down some scattered thoughts on the conference. Things I learnt, themes I saw emerging, new science appearing and old science being adapted.

And then I thought these thoughts might just about be useful enough to others to warrant publishing, and so here is a brief post along those lines. I’ve tried to categorise into somewhat arbitrarily chosen categories, but really this is just a disparate collection of things that popped into my head as I read through my notes. Enjoy 🙂

{{< notice >}}
Got any thoughts, comments, criticisms? Please do comment below - you’ll need a GitHub account, but seeing as we’re all moving towards FAIR data and software, you’ll probably want a GitHub account anyway to start open-sourcing your scripts, data analysis and models 😉
{{< /notice >}}

{{< bannerimage src="panorama.jpg" alt="Photo of a lake surrounded by mountains with a large coniferous tree in the foreground" >}}

## Exposure and release modelling

- Catchment-based spatiotemporal exposure and fate models are being developed by multiple groups. Mainly for surface waters, but some spatial soil erosion modelling too.
- Environmental properties are as important as polymer properties in determining fate (which mirrors what I always say about nanomaterials).
- Emissions models are being developed, for [specific countries](https://doi.org/10.1021/acs.est.9b02900) and [globally](https://doi.org/10.1021/acs.est.9b02900). Results for Switzerland show [tyre wear](https://doi.org/10.1016/j.envpol.2019.113573) and [soils](https://doi.org/10.1021/acs.est.9b02900) are very important.
- [Snowmelt is an important transfer of microplastics](https://doi.org/10.3389/frwa.2022.958130) to surface waters in snowy urban areas.
- Fragmentation and additives are not included in most exposure models, but there is work to change that (Cefic LRI projects [ECO58](https://cefic-lri.org/projects/eco-58-comprehensive-additive-release-and-bioaccessibility-model-for-risk-assessment-of-micro-and-nano-plastics-in-the-environment/) and [ECO59](https://doi.org/10.5281/zenodo.7035673)).
- Screening-level, unit-world models are also seeing some development, e.g. adaption of SimpleBox4nano to plastics, and (not presented) [Cefic LRI project ECO56](https://cefic-lri.org/projects/eco-56-utopia-development-of-a-multimedia-unit-world-open-source-model-for-microplastic/).
- Long-range transport is an important process in moving plastics to remote areas, and there is also a need for long-range transport models. [Cefic LRI project ECO57](https://cefic-lri.org/projects/eco-57-%ce%bcplanet-microplastic-long-range-transport-assessment-and-estimation-tools/) is addressing this.
- Lots of exposure modelling is going on separately (including work not presented at Microplastics2022) - we should talk more to avoid reinventing the wheel! Is there justification and desire for a microplastic exposure modellers initiative/platform/network?

## Analytics

- QA/QC is super important! Define your analytical window (e.g. what size range can you detect), report controls (including positive controls).
- The field of analytical development is very active. Techniques commonly employed in other fields (e.g. cathodoluminescence) should be considered for use with micro- and nanoplastics.
- Machine learning is playing an ever-important role in data analysis towards routine analysis, e.g. in finding microplastics in FTIR and Raman spectroscopy data. Given the ever-increasing amount of analytical data available on microplastics, data science in general is becoming more and more important.

## Fragmentation, additives, shape and size

- Plastic fragments aren’t spherical, and there is work ongoing to quantify the distribution of shapes present in the environment.
- Because of this, think about the relevance of using polystyrene spheres in your experiment, or assuming sphericity in your models ([oops 😳](https://github.com/microplastics-cluster/fragment-mnp/blob/dc0c29edaccf1845d0d998c7f37a20a6eb649112/src/fragmentmnp/fragmentmnp.py#L120-L124)). Models could easily use form factors to have more realistic representations of shape.
- Settling of non-spherical particles is slower than Stokes' Law. Settling of fibres is *much* slower. Even alternative adaptations of Stokes’ Law (e.g. [Zhiyao, 2018](https://doi.org/10.1016/S1674-2370(15)30017-X)) don’t work well for fibres.
- Instead of using binned size, shape and density classes to describe your polymer, [consider using continuous probability distributions](https://doi.org/10.1021/acs.estlett.9b00379) to fully reflect the continuous nature of plastics in the environment (again, [oops 😳](https://github.com/microplastics-cluster/fragment-mnp/blob/dc0c29edaccf1845d0d998c7f37a20a6eb649112/src/fragmentmnp/examples.py#L20)).
- Volume and surface area should be reported as part of risk assessments.
- Additive leaching can happen over decades (e.g. [leaching of phthalates from PVC in the aquatic environment](https://doi.org/10.1021/acs.est.2c05108)). Don’t assume additives quickly leach from polymers!
- The variety of additives used is very broad, and the effect of additives on polymer characteristics (and therefore exposure and hazard) can be considerable. Describing a polymer simply by its type can be misleading - additive content (and intended action of the additive) must be defined.
- Our [molecular understanding of](https://doi.org/10.1021/acs.est.0c07718) [fragmentation is advancing](https://doi.org/10.1016/j.scitotenv.2022.154035), with most work suggesting the formation of micro- and nanofragments by surface erosion is a key process.
- Biodegradable plastics are an important part of the toolkit to tackle plastic pollution, but in limited applications. They aren't a panacea. Education is needed to manage waste streams.

## General

- Don’t forget the [lessons from the](https://doi.org/10.1038/s41565-020-0742-1) [nano world](https://doi.org/10.1021/acs.est.6b04054).
- Are polymer particles like sediments? Opinion is divided! (My take: Similar to the nano-to-microplastics crossover, there are enough similarities to be useful, so we should figure out where we can use previously developed approaches from the sediment world, paying careful attention to the many places where plastics deviate from sediment behaviour.)
- It doesn't matter that we don't yet have a holistic plastics risk assessment - plastics shouldn't be in the environment and that’s good enough reason to act (reduce, mitigate, etc).
- Don't forget lots of the planet doesn't have managed waste, including certain communities in developed countries where waste dumping and burning is commonplace. Global treaties need funding and education for communities to properly manage plastic waste.
- Plastics are important, we can't just ban them.
- Public concern and societal values are very important. Companies are keen to act on public concerns.

## Community and conference

- The environmental microplastic community doesn’t have a central network like the NanoSafety Cluster was for nanoscience. Human microplastics research has [CUSP](https://cusp-research.eu/). Do we need one?
- Also see previous point about overlapping exposure modelling work. Do we need a platform to discuss this work?
- Running a conference #1: Long breaks are great! A 0830-1900 agenda is completely okay with 1.5 hour lunches, 50 min coffee breaks and 2 hour poster sessions. Long breaks = much more time for networking. I thought 0830-1900 would burn me out, but with the breaks it felt really relaxed and I left feeling much less drained (and much more energised about the science) than most other conferences.
- Running a conference #2: 20 minutes per talk is *so* much better than 15 minutes. 20 minutes gives time for an in-depth 15 minute talk and then 5 minutes for a proper Q&A session. 15 minutes means people overrun and have 1 or 0 minutes for questions.

{{< figure src="group-photo.jpeg" alt="Photo of a large group of people posing for a group photo infront of a lake" >}}
