# Abstract

Sea ice forms from seawater at the interface between the ocean and the atmosphere. Its formation plays a key role for vertical exchange
of salt and heat within the upper ocean and for the global thermohaline circulation, and its melt influences near-surface stratification
of the polar and surrounding seas. The presence or absence of sea ice has large implications for human-related activities in the 
polar regions (especially the Arctic), including shipping, resource exploitation (incl. fishing) and the lifestyle of indigeneous 
communities.

The area fraction of sea ice is also commonly named the {term}`Sea Ice Concentration` (SIC). It is a unitless number (the ratio of two
areas) and is reported either as a number in the range [0;1] or as a percentage in the range [0-100%]. SIC is by far the most important
variable when it comes to weather, ocean, and sea ice forecasting in the polar regions, and virually all operational centers assimilate
satellite-based products of {term}`SIC` to guide there forecasts.

Sea ice concentration has been derived from satellite passive microwave imagery for the over 40 years. The U.S. passive microwave missions
{term}`SMMR`, {term}`SSM/I`, and {term}`SSMIS` have been the workhorse of climate monitoring of sea-ice concentration / extent / area since
the late 1970s. In the last two decades, the Japanese missions {term}`AMSR-E` and {term}`AMSR2` have allowed higher spatial resolution but
at the cost of larger retrieval uncertainties (using high-frequency microwave imagery that the atmosphere between the surface and the satellite
perturbe more). The {term}`Copernicus Imaging Microwave Radiometer` (CIMR) will bring higher resolution at the low-frequency "window" microwave
channels, allowring high resolution and high accuracy for many operational applications.

Key assets of CIMR in terms of sea-ice concentration monitoring are the high resolution (4-5 km) capabilities at Ku- and Ka-band, two microwave
frequencies that are at the core of most state-of-the-art sea-ice concentration algorithms today and medium resolution (15 km) capabilities at
C-band, a channel where the the atmosphere is mostly transparent and the sea-ice emissivities do not vary too much with sea-ice type. In
addition, the swath width and especially the "no hole at the pole" capability will improve coverage of sea ice monitoring in the Arctic. The
availability of a forward and a backward scan should be exploited.

CIMR has a requirement to provide a set of Level-2 products with 1 hour latency (see {term}`Near Real Time 1H`), the first of which SIC, for
supporting safety at sea in Arctic regions. This requirement is translated in a dedicated {term}`SIC1H` Level-2 product that has to achieve
high resolution within a short latency. This might lead to reduced accuracy wrt to the nominal {term}`SIC3H` product. Typically, it means
the SIC1H algorithm should focus on Ku- and Ka-band imagery and potential skip some parts of the {term}`SIC3H` algorithm, e.g. atmospheric correction
of the brightness temperatures or deriving uncertainty estimates.

Sea Ice Edge ({term}`SIE`)  is a binary product derived from SIC to quickly indicate where there is sea ice and where the ocean is free for ice. It is
typically derived from a simple binary test involving a SIC threshold, classically 15% SIC (all regions with SIC larger than the threshold are considere
ice-covered). From the binary product, one can derive the contour polygon of the ice cover.
