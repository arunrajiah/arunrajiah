# Catalyst Grant 2026: an agent pipeline turning machine wildlife detections into citable research data

Arun Rajiah, independent open source maintainer, Chennai, India. github.com/arunrajiah

## 1. THE PROBLEM

Two groups share it.

Community sensor operators. Tens of thousands run continuous wildlife sensors outside any institution: BirdNET-Pi and BirdNET-Go acoustic stations in gardens, farms and reserves, and camera traps run through AI classifiers such as SpeciesNet. One station produces hundreds to thousands of species level detections a day, every day, and that output sits in a local SQLite file or a vendor platform. To contribute it to GBIF, the global biodiversity record, an operator must hand export, reshape to Darwin Core, write dataset metadata, find a publisher and run an IPT. It is days of unfamiliar work, repeated on every refresh, so the real cost is that it never happens.

Biodiversity researchers and aggregators. They need this data and cannot trust most of what reaches them, because the provenance has been flattened away: a published record rarely says which model produced it, which version, at what confidence, or whether a human checked it. A machine false positive then enters the scholarly record as an unqualified occurrence. That is a research integrity problem, and it worsens as classifiers get cheaper.

The failure is continuous, not occasional: the stream is generated every minute and almost none of it becomes citable science.

## 2. YOUR WORKFLOW

Trigger: new detections appear at a station, on a schedule the operator sets.

1. Ingest. Read BirdNET-Pi SQLite, the BirdNET-Go API, SpeciesNet JSON or Raven tables and normalise to WDX, the Wildlife Detection Exchange format I published this year.
2. Validate against the WDX JSON Schema. Repair malformed input where the repair is unambiguous, quarantine it where it is not.
3. Resolve taxon names against the GBIF Backbone and eBird/Clements. Unresolved names are flagged, never guessed.
4. Gate. Apply the operator's policy: minimum confidence per species, coordinate generalisation for home addresses following Chapman (2020), exclusion of rejected records, suppression of sensitive taxa.
5. Assemble a Darwin Core occurrence dataset, basisOfRecord MachineObservation, with dataset metadata drafted by the agent.
6. Release. Show the operator a diff of what would publish and what was held back. On approval, publish with a DOI and the audit log attached.

Steps 1 to 5 run autonomously, including retries, repair and metadata drafting. A person holds three things: the release gate, the quarantine queue, and any taxon the agent could not resolve. The agent never invents an observation, never promotes an unreviewed detection to reviewed, and never publishes without an explicit human release.

Where it lives. On the input side, inside BirdNET-Pi and BirdNET-Go stations, reached through BirdEcho, my companion app for those stations, and SpeciesNet Studio, my camera trap review interface. Operators open both daily. On the output side it targets the GBIF IPT and EarthRanger through Gundi.

Outcome metric: records per month reaching GBIF with complete machine readable provenance, and the number of distinct stations publishing at all. For community acoustic monitoring both are zero today.

## 3. TRUST, AUDIT AND GOVERNANCE

Every published record carries its full chain: which sensor, which model, which version, what confidence, whether a human confirmed it, and every transformation applied on the way. The audit log is append only and ships with the dataset, so a reader can reproduce or discount any record without asking me.

Provenance. WDX (DOI 10.5281/zenodo.22200864) makes classifier identity, version, confidence and review status first class fields rather than free text, and carries human review separately from machine identification so consumers can filter on either. Darwin Core has no confidence term at all, a gap its own tracker closed unresolved in 2019, so my crosswalk states plainly what survives the mapping and what does not.

When it is uncertain or wrong. Below threshold and unresolvable records are quarantined rather than deleted, because a rejection is evidence, and the rejection set can ship alongside the dataset so a user can measure the filter instead of trusting it. I tested this deliberately: given a detection whose taxon resolved to nothing, and a second geolocated at a private residence, the pipeline refused to publish the first and generalised the coordinate on the second rather than doing its best and moving on. Accountability sits with the named operator who releases the dataset; the agent's authority is bounded and every use of it logged.

I have shipped this pattern in production: OdooPilot, my AI assistant for the Odoo ERP, is on the Odoo App Store for 17.0 and 18.0 and has passed four store security audits, and every write it makes is a guided action a named human confirms.

## 4. TEAM

A solo open source maintainer in Chennai, India, self taught, 31 public repositories. No co-founders, no institutional backing.

I picked this problem because I hit it myself. I maintain BirdEcho, a companion app for BirdNET-Pi, BirdNET-Go and BirdWeather stations; wildecho-api, self hosted identification on Google's Perch 2.0 model, CPU only and Dockerised; and SpeciesNet Studio. I wrote WDX because nothing existed to carry a detection honestly from my stack into a standard dataset.

Expertise: bioacoustic and camera trap pipelines end to end, from inference through review to publication, plus production experience building audit first agents where a wrong autonomous write is expensive.

Practitioner review is already shaping v0.2, from a University of Florida engineer building underwater video annotation pipelines and an Imperial College London researcher working on shark classification from BRUV video, both engaging through WILDLABS and now through the specification's public issues.

## 5. WHERE YOU ARE TODAY

Prototype, with several components already in real use.

- WDX specification, JSON Schema, Darwin Core crosswalk and worked examples for BirdNET, Perch and SpeciesNet producers: github.com/arunrajiah/wildlife-detection-exchange, DOI 10.5281/zenodo.22200864
- BirdEcho, in use by station operators today: github.com/arunrajiah/birdecho
- wildecho-api and SpeciesNet Studio, working inference and review components: github.com/arunrajiah

What is not built is the connective pipeline itself, though its crosswalk is published, its schema validates, and every stage exists somewhere in my stack today.

How I tested the idea: I took the specification to the operator communities rather than writing it in private, posting in BirdNET-Pi discussion 649, BirdNET-Go discussion 4227 and on WILDLABS. The requirements that came back, sequence grouping for camera trap bursts and marine and underwater fields, are now open issues driving v0.2.

## 6. ALTERNATIVES AND COMPETITORS

Doing nothing is the dominant alternative, and what nearly every operator does.

Camtrap DP with the GBIF IPT serves institutional camera trap studies that employ a data manager; passive acoustic monitoring has no equivalent at all, which is the cleanest gap in the field. BirdWeather and Wildlife Insights aggregate community data on their own terms, with the operator's raw stream held inside the platform. General ETL tools can do the transformations but assume a data engineer, exactly the person a home station does not have.

The gap is checkable rather than asserted: EarthRanger's Gundi integration catalog, the nearest thing conservation technology has to a common bus, lists no bioacoustic and no AI camera trap source.

How I differ: I treat the machine detection, carrying its model version, confidence and review state, as the unit that must survive intact to publication, and I aim the workflow at the operator rather than at an institution's data office.

## 7. WHERE THIS GOES

Community sensors become a first class source of research data. A reserve in Tamil Nadu or a birder in Helsinki runs one open tool, and their verified detections flow into GBIF with honest provenance, so researchers can cite datasets whose every record traces to a sensor, a model version and a review decision. The pattern generalises wherever machine observations must become trustworthy evidence.

This is not a commercial product at its core. The specification and the pipeline stay open source under community governance, because a standard one person can charge for is not a standard. Any revenue comes from hosted ingest for organisations that would rather not self host, and from paid integration work for platforms, never from the operators whose participation is the point.

## 8. FIT WITH DIGITAL SCIENCE

Lifecycle position: discovery and research integrity. Audience: researchers and the institutions publishing on their behalf, with the resulting datasets landing where funders and industry can see them.

Why I want to work with Digital Science. This manufactures well formed, DOI bearing, provenance complete datasets from a source the research information ecosystem cannot currently see, which is the Figshare publishing model applied one step upstream. The integrity agenda is not bolted on to fit the brief, it is the design centre: machine generated evidence should enter the scholarly record with its uncertainty and lineage intact rather than laundered into false confidence. An agent that publishes only what it can prove is that principle made operational.

## 9. BUDGET

Up to GBP 25,000: GBP 17,000 for twelve months of development taking the pipeline from prototype to a released, documented tool; GBP 2,500 for hosting and continuous integration of a reference ingest endpoint and public demo datasets; GBP 3,500 for review bounties paid to operators who run it on their own stations and audit the output; GBP 2,000 for documentation, accessibility and security review. It buys what I cannot do unpaid: a sustained year, plus independent verification by real operators.
