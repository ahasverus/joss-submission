---
title: 'forcis: An R package for accessing, handling and analysing the FORCIS Foraminifera database'
tags:
- r
- database
- planktonic foraminifera
- biodiversity
- species abundance
- data visualisation
date: "9 June 2025"
output: pdf_document
authors:
- name:
    given-names: Nicolas
    surname: Casajus
  orcid: "0000-0002-5537-5294"
  equal-contrib: no
  corresponding: yes
  affiliation: 1
- name:
    given-names: Sonia
    surname: Chaabane
  orcid: "0000-0002-4653-8610"
  equal-contrib: no
  corresponding: no
  affiliation: "1, 2, 3"
- name:
    given-names: Xavier
    surname: Giraud
  orcid: "0000-0001-5067-8176"
  equal-contrib: no
  corresponding: no
  affiliation: 2
- name: 
    given-names: Thibault
    dropping-particle: de
    surname: Garidel-Thoron
  orcid: "0000-0001-8983-9571"
  equal-contrib: no
  corresponding: no
  affiliation: 2
- name: 
    given-names: Mattia
    surname: Greco
  orcid: "0000-0003-2416-6235"
  equal-contrib: no
  corresponding: no
  affiliation: 4
bibliography: paper.bib
affiliations:
- name: "Fondation pour la Recherche sur la Biodiversité (FRB-CESAB), Montpellier, France"
  index: 1
- name: "Aix-Marseille Université, CNRS, IRD, INRAE, CEREGE, Aix-en-Provence, France"
  index: 2
- name: "Department of Climate Geochemistry, Max Planck Institute for Chemistry, Mainz, Germany"
  index: 3
- name: Institute of Marine Sciences (ICM), CSIC, Barcelona, Catalonia, Spain
  index: 4
---

# Summary

`forcis` is an R package designed to streamline access to the recently published FORCIS (Foraminifera Response to Climatic Stress) database (**REF**). This database provides one of the most comprehensive collections of global planktonic foraminifera living census data, comprising over 163,000 samples collected via various sampling devices between 1910 and 2018.

This package enables users to download the database directly into an R environment, filter and select relevant data, convert species counts across formats, and visualise the results.


# Statement of need

...


# Main features

To facilitate efficient management and analysis of the FORCIS database, the `forcis` R package provides a comprehensive set of features fully described in the package vignettes, where users can find extensive documentation and tutorials on the major features of the package. The recommended workflow and the relevant main functions are illustrated in Fig. 2.


## Download and import FORCIS database in R

The `forcis` R package contains functions that simplify downloading and importing FORCIS datasets from Zenodo (https://doi.org/10.5281/zenodo.7390791). The FORCIS database's most recent version can be retrieved using the function `download_forcis_db()`. 

```r
# Create a data/ directory in the current directory ----
dir.create("data")

# Download the latest version of the FORCIS database ----
download_forcis_db(path = "data", timeout = 300)
```


The `read_*_data()` function family helps users in importing dataset specific to a particular sampling device, enabling focused analyses.


```r
# Import plankton nets data (previously downloaded) ----
net_data <- read_plankton_nets_data(path = "data")
```


Once the data is imported in R, users can reduce the dataset to include only the metadata they are interested in by using the function `select_forcis_columns()`.


## Harmonising taxonomy

To utilise most features of the `forcis` R package, users need to specify the taxonomic framework they wish to apply (Fig. 2). The FORCIS database includes counts at three different taxonomic levels: Original Taxonomy (OT), Lumped Taxonomy (LT), and Validated Taxonomy (VT). For a detailed explanation of the differences between these three taxonomic levels, we refer the reader to the FORCIS data descriptor (Chaabane et al., 2023). For selecting the taxonomic framework of choice, the users can use the function `select_taxonomy()` following the example below:


```r
# Select a taxonomic framework ----
net_data_vt <- net_data |>
  select_taxonomy(taxonomy = "VT")
```

## Filter data

After selecting the taxonomic framework, the `forcis` R package offers multiple functions to efficiently subset the FORCIS datasets. Users may be interested in analysing community structure at a specific time, or location, or even examining the counts of species of interest. Given the wide range of potential research questions, we have implemented six filtering functions within the `filter_by_*()` function family, allowing users to customise data extraction according to their investigation needs (Fig. 2 and Table 1).


```r
# Filter data by year(s) ----
net_data_sub <- net_data_vt |>
  filter_by_year(years = 1992)

# Filter data by spatial bounding box ----
net_data_sub <- net_data_vt |>
  filter_by_bbox(bbox = c(45, -61, 82, -24))

# Filter data by ocean name ----
net_data_sub <- net_data_vt |>
  filter_by_ocean(ocean = "Indian Ocean")

# Filter data by species ----
net_data_sub <- net_data_vt |>
  filter_by_species(species = "n_pachyderma_VT")
```


## Transform data

The `compute_*()` function family allows users to convert FORCIS data between raw abundance, number concentration, and relative abundance, enabling them to use the units that best suit their analyses and facilitating comparison between the FORCIS data and their own.
These functions utilise sample metadata to perform unit conversions. Specifically, conversions between raw abundance and number concentration in the `forcis` R package are calculated for each taxon using the following equations:

$$C_{number} = \frac{N_{raw}}{V_{filtered}}$$

where $C_{number}$ is the number concentration, $N_{raw}$ is the raw abundance (count of individuals), and $V_{filtered}$ is the volume of water filtered (in m^3 or L, depending on the dataset).

$$Frequency = 100 \cdot \frac{N_{raw}}{N_{total}}$$

where $Frequency$ is the relative abundance (in percentage), $N_{raw}$ is the raw abundance (count of individuals) of a given taxon, and $N_{total}$ is the total raw abundance (sum of all individuals in the sample or subsample).

The users can decide whether to convert counts at a sample or subsample level (see Chaabane et al., 2023) as the `compute_*()` functions propose the `aggregate` argument. If `aggregate = TRUE`, the function will return the transformed counts of each species using the sample as the unit. If `aggregate = FALSE`, it will re-calculate the species' abundance by subsample. 

The output of these functions is a table in a long format as well as a message reporting the amount of data that could not be converted because of missing metadata required to perform the transformation.


## Visualisation

The `forcis` package also includes multiple functions to visualise the spatial distribution of samples selected by users. The `ggmap_data()` function generates publication-ready maps, displaying sample locations at a global scale (Fig. 3a). Additionally, users can visualise sample records by various time units (season, month, year) and by depth, using the functions from the `plot_record_by_*()` function family (Fig. 3b-d).
These functions can be seamlessly combined with the `filter_by_*()` family of functions, allowing users to customise their sample selections according to their specific research needs (see Box 1).

```r
# Map raw net data ----
ggmap_data(net_data)

# Plot number of records by year of sampling ----
plot_record_by_year(net_data)

# Plot number of records by month of sampling ----
plot_record_by_month(net_data)

# Plot number of records by depth of sampling ----
plot_record_by_depth(net_data)
```

# Case study

To illustrate the capabilities of the `forcis` R package, we investigated the distribution data of planktonic foraminifera species _Neogloboquadrina pachyderma_ between 1970 and 2000 in the North Atlantic Ocean.

```r
# Prepare data ----
net_data <- read_plankton_nets_data(path = "data/") |>
  select_taxonomy(taxonomy = "VT") |>
  select_forcis_columns() |>
  filter_by_ocean(ocean = "North Atlantic Ocean") |>
  filter_by_year(years = 1970:2000) |>
  filter_by_species(species = "n_pachyderma_VT") |>
  dplyr::filter(n_pachyderma_VT > 0)

# Map N. pachyderma records ----
ggmap_data(net_data)

# Plot number of records by month ----
plot_record_by_year(net_data)
```


# Opportunities and future directions

The forcis R package has been designed to streamline access to the FORCIS database by enabling users to seamlessly download data directly into an R environment. It provides robust tools for filtering and selecting relevant data, converting species counts across multiple formats, and visualising the results, making it an indispensable resource for researchers looking to efficiently manage and analyse FORCIS data.  This package is open to external collaborations, and we welcome contributions to enhance its features.

One potential development for forcis could be the addition of analytical features, such as normalising abundance data to a standardised mesh size of over 100 µm (Chaabane, et al., 2024b). Furthermore, forcis could expand its analytical tools to include dynamic visualisations, such as interactive maps and time-series animations, allowing users to explore complex data more intuitively.  In addition, forcis could facilitate the integration of new planktonic foraminifera census data into the database by sourcing observations from environmental and biodiversity platforms such as OBIS (https://obis.org), GBIF (https://gbif.org), PANGAEA (https://www.pangaea.de) and  EcoTaxa (https://ecotaxa.obs-vlfr.fr). 

To broaden its user base, the forcis R package is aiming to continuously update the user-friendly guides and tutorials, as well as organise outreach initiatives to integrate the forcis R package into workshop sessions at FRB-CESAB. This will help expand data collection efforts, ensuring that the forcis R package remains a useful tool for the marine research community in the years to come.

The tool we present underscores that simply releasing data is not enough to ensure its reusability. To truly uphold FAIR principles, data products must be accompanied by the tools and software needed to access, process, and interpret them. We advocate for a broader commitment to developing and sharing such tools as an integral part of open science.



# Acknowledgements

The FORCIS project is supported by the French Foundation for Biodiversity Research (FRB) (https://www.fondationbiodiversite.fr) through its Centre for the Synthesis and Analysis of Biodiversity (CESAB) (https://www.fondationbiodiversite.fr/en/about-the-foundation/le-cesab/) and co-funded by INSU LEFE program and the Max Planck Institute for Chemistry (MPIC) in Mainz. M.G. was supported by a Juan de la Cierva-formacion 2021 fellowship (FJC2021–047494-I/MCIN/AEI/10.13039/501100011033) from the European Union “NextGenerationEU”/PRTR and by the Beatriu de Pinós  programme (2022 BP 00209) funded by the Direcció General de Recerca (DGR) del Departament de Recerca i Universitats (REU) of the Government of Catalonia. In addition, his work received support from the French government under the France 2030 investment plan, as part of the Initiative d’Excellence d’Aix-Marseille Université (A*MIDEX AMX-20-TRA-029). The authors would like to thank Beatriz Milz, Scott Chamberlain and Air Forbes for theirs valuable comments during the peer review process in rOpenSci (https://ropensci.org).




# References

...
