---
layout: post
title: Computational Materials Discovery - a primer
---

![Graphene Salination](/images/Graphene_salination.jpg)

<i>Desalinating water using Graphene. Image from article in <a href = "https://www.economist.com/babbage/2013/04/04/allo-allo">the Economist</a></i>

# Table of Contents  
1. [Introduction](#intro)  
2. [Electronic Configuration - an intuitive explanation](#elec-config)  
3. [Average Electronic Configuration](#av-config)  
4. [Materials Discovery - a case study](#case-study)  
        [Relevant questions to investigate](#questions)  
        [Step 1. Gather Data](#gather)  
        [Step 2. Assess Data](#assess)  
        [Step 3. Clean and prepare data for analysis](#clean)  
        [Step 4. Data analysis, modeling and visualization](#analysis)  
5. [Conclusions](#conclusions)  

<a name="intro"></a>  
## Introduction  

<i>Materials make the world</i>  
  
Does this statement seem corny and hyperbolic at first sight? I'd have to agree. While that may be true, I also hold that the statement is a self-evident truth. Still skeptical? Good. You have reasons to read on.

To make my case, let's run through a typical morning routine. You wake up, you brush your teeth using a plastic tootbrush with nylon bristles. You use a ceramic toilet. The stainless steel showerhead pours cleansing water on you. You eat breakfast while checking your phone made of a metallic alloy and coated with unbreakable Glass. Oh, and that phone contains a microprocessor chip that houses within it materials comprised of greater than 30 elements in the periodic table. You then leave for work in a car that likely has a frame strengthened with composites. You spend your day staring at an LCD/LED screen that receives display instructions from a slightly larger microprocessor chip that...is also made of materials that run the gamut of elements in the periodic table.

I won't belabor the point further. Hopefully you've caught on to my drift by now. We take these materials and the conveniences they enable for granted. This is as it should be. We shouldn't spend each moment of our waking lives worrying about how things were made. But I think it is useful to pause and reflect upon how these materials get into objects of daily use.

![Materials Continuum](/images/materials_continuum.jpg)
<i>Figure source: <a href = "https://www.mgi.gov/sites/default/files/documents/materials_genome_initiative-final.pdf">Materials Genome Initiative</a></i>

A promising material candidate is identified for an application. This step is then succeeded by several stages - development, optimization, design, certification, manufacturing and finally, deployment. The <a href = "https://www.mgi.gov/sites/default/files/documents/materials_genome_initiative-final.pdf">Materials Genome Initiative</a> estimates a timeframe of 10 to 20 years for the above process. Not to mention the fact that the discovery process itself is prone to trial-and-error and extremely time consuming. 

Advances in computation have enabled the creation of a new field - <a href="https://iopscience.iop.org/article/10.1088/1361-6463/aad926/pdf">Computational materials discovery and design</a>  - that aims to make the process above more systematic and less time-consuming. The essential idea in computational materials discovery is to replace the costly and time-consuming trial-and-error phase of discovery driven by existing chemical intuition and brute-force experimentation in favor of highly-accurate and targeted data-driven approaches.

In other words, before conducting physical experiments, one steps back, does a detailed analysis of existing materials databases or generates new data based on the fundamental physics of materials, and generates a list of promising materials suited for targeted physical properties. 

In this post, I aim to give the reader a brief but hopefully informative snapshot of the computational materials discovery process. In the interest of simplicity, I use the <a href='https://en.wikipedia.org/wiki/Electron_configuration'>Electronic Configuration</a> of atoms to group materials and find materials having properties similar to industrially relevant materials.

At the outset, I caution that the post is not intended to be exhaustive conceptually. Instead, I hope to convey a sense for how the problem of materials discovery can be tackled using a principled, data-based approach using computational tools that are free and easily accessible.

The rest of this post is organized as follows. First, I attempt to provide an intuitive explanation of the concept of average electronic configuration of materials. I then describe the use of two Python software packages - <a href = "http://http://pymatgen.org/">pymatgen</a> and <a href = "https://hackingmaterials.github.io/matminer/">matminer</a> - in creating a database of the valence electron configuration of a large number of inorganic materials. I then outline specific questions in the context of materials similarity and discovery that can be answered by performing a detailed analysis of database. Each subsequent section provides answers to these questions in the form of visualizations and tables. The last section provides key take-home messages from the analysis and some suggestions for expanding and improving upon the analysis in this post. Accompanying code for this post can be found at my repository on <a href = "https://github.com/g-hegde/material-similarity">GitHub</a>. 

<a name="elec-config"></a>
## Electronic configuration - an intuitive explanation  

The <a href='https://en.wikipedia.org/wiki/Electron_configuration'>Electronic Configuration</a> of atoms describes how electrons are distributed in an atom. In a broad sense, atoms are comprised of

* A central core called the nucleus, which contains neutrons and positively-charged protons  
* Negatively-charged electrons that orbit the nucleus

Electrons orbiting the nucleus do so in spatial configurations called orbitals at multiple energy levels (shells). Orbitals come essentially in 5 different flavors - s, p, d, f and g. Each orbital can hold a maximum of 2 electrons. At each energy level, one has a combination of s and/or p, d, f and g orbitals. s-orbitals are spherically symmetric and there is consequently only one s orbital for each energy level. There are 3 and 5 p and d orbitals respectively for energy levels in which these orbitals exist. Valence electrons are electrons that occupy the outermost energy levels. These are less-bound to the nucleus and are consequently freer to move through the body of the material. Valence electrons and their resulting configuration is what determines bonding and chemical properties of a material. The picture below shows the Valence Electronic Configuration (VEC) of elements in the periodic table.

![Periodic_Table.jpg](/images/Periodic_Table.jpg)  
<i>Image from <a href = "https://opentextbc.ca/chemistry/chapter/6-4-electronic-structure-of-atoms-electron-configurations/">OpenBC Textbook</a></i>

The VEC of materials is one of their most fundamental physical properties. Knowledge of the VEC of atoms in materials can help us understand and predict a wide range of their physical and chemical properties. For instance, knowledge of the VEC of Silicon 3s<sup>2</sup>3p<sup>2</sup> explains why it is a semiconductor. Or why Copper, Silver and Gold ns<sup>1</sup>(n-1)d<sup>10</sup> are metals.

![Si_VEC.png](/images/Si_VEC.png)  

<i><a href = "http://www.chem4kids.com/files/elements/014_shells.html">Silicon Electronic shell levels</a>. The Valence Electronic Configuration of Silicon is 3s<sup>2</sup>3p<sup>2</sup>. The outermost shell (3) contains a filled s orbital and two partially filled p orbitals</i>  

Developing a deep understanding of electronic configuration requires extensive knowledge of atomic physics and chemistry. This is because, at a fundamental level, the behavior of all materials is governed by the rules of <a href="https://en.wikipedia.org/wiki/Quantum_mechanics">Quantum Mechanics</a>. Needless to say, this is beyond the scope of a single blog post. It is useful instead to focus on understanding how the VEC changes from one atom/material to another.

The periodic table above shows the VEC of all elemental materials (The terms elemental/unary/mono-atomic will be used interchangeably here). For compound materials such as binary materials (materials comprised of two elements) and ternary materials (three elements), such a straightforward comparison is not possible.

<a name="av-config"></a>  
## Average electronic configuration  

For materials containing multiple elements, a simple tweak to valence electronic structure is obtained as follows. An <a href = "http://chemed.chem.purdue.edu/genchem/topicreview/bp/ch13/unitcell.php">Unit Cell</a> of a material is a repeatable unit of atoms of a material. When this unit is repeated uniformly in all spatial dimensions, we obtain a crystal of the material. By summing the VEC of each atom in the Unit Cell and dividing through by the number of atoms in the unit cell we obtain the average VEC.

Having seen what the average VEC is, we can put this knowledge to use in a simple materials discovery case study.

<a name="case-study"></a>  
## Materials Discovery - case study   

We have a material that we seek to replace with another cheaper, equally performing alternative. As a Computational Materials Discovery scientist, it is your responsibility to create a database with relevant VEC information for all possible elements and compounds that may be suitable replacements for this material. You are then required to clean this database to ensure the fidelity of all the data in the database. After this, you are required to perform unsupervised machine learning to see which materials group together. The set of materials closest to the target material can then be pushed up the chain for further consideration as replacement candidates.

<a name="questions"></a>  
## Relevant questions to investigate    
* When materials are represented on the basis of their average Valence Electronic Configuration (VEC), what is the optimum number of clusters that they can be grouped into?  
* What materials are represented by cluster centers? Alternately, what unary material/materials are closest to cluster centers? 
* Does the grouping make intuitive sense - i.e. are the cluster centers sufficiently far away from each other on the basis of chemical intuition?  
* Given a target material, say Copper, what is the material (unary, binary and ternary respectively) whose VEC most closely resembles that of Copper? Performing such an analysis could be useful for finding replacements for <a href = "https://en.wikipedia.org/wiki/Copper_interconnects"> Copper interconnects</a> in microprocessors.
* Given a target material, say Copper, what is the (binary and ternary) nitride material whose average VEC most closely resembles that of Copper?  Performing such an analysis could be useful for finding suitable diffusion barrier materials for Copper interconnects.

<a name="gather"></a>  
## Step 1. Gather Data  
The first step in the process is to create a dataset that is suitable to answer the questions posed above. What kind of datashould we gather? At minimum we need the VEC of every material we plan to investigate. How many materials should we investigate? This is really a subjective choice, but one can choose to investigate as many materials as one has access to. The <a href = "https://www.materialsproject.org/about">Materials Project</a> provides query-based access to material properties of over 100 thousand inorganic materials.  

For this blog post, I decided to query the Materials Project using the API functionality provided by the <a href= "http://pymatgen.org">pymatgen</a> library. After filtering for materials that were radioactive, inert or highly reactive, I first gathered material identifiers of over 50 thousand inorganic materials.  

The <a href="https://hackingmaterials.github.io/matminer/">matminer</a> package is a library that can be used to perform data mining of materials. It provides routines to query various materials databases, featurize existing databases and works natively within the Pandas DataFrame format. This makes the package extremely useful for this case study.  

I used the <a href = "https://hackingmaterials.github.io/matminer/matminer.featurizers.html#matminer.featurizers.composition.ValenceOrbital">ValenceOrbital Featurizer </a> in matminer to obtain the VEC of each material. For the questions above, I decided to limit myself to the 's', 'p' and 'd' orbital VEC since this covers most elements of practical use.  

<a name="assess"></a>
## Step 2. Assess data

Having created a dataset for analysis, one can go about analyzing the data to get an intuitive feel for the dataset. This includes knowing the range of each variable in the dataset, the distribution of values and outliers if any. The figure below shows a distribution of s, p and d electrons for the data set.  

![VEC Distribution](/images/spd_dist.png)  

The distribution of s-electrons and p-electrons are almost diametrically opposite - The s distribution is heavily left-skewed, while the p-distribution is heavily right-skewed. d-electrons have a comparatively more even distribution with a few spikes.  

What are the insights that we get from this assessment? First, left-skewed s data indicates that materials with unoccupied s-orbitals are far and few in the dataset. This is potentially disadvantageous if we are looking for exact replacements to materials that have unoccupied s orbitals - e.g. some materials from Group 1 and periods 4,5, and 6 in the periodic table and their combinations.  

The exact opposite is true of p orbitals - the number of materials with unoccupied p orbitals is large, while the number of materials with occupied p-orbitals is small. Inspecting the Periodic table above, the reason for this imbalance becomes clear - we excluded inert gases (which are pretty useless for our case study) and the highly reactive Fluorine-group elements. It also means that the dataset is skewed towards more metallic elements (and their compounds) on the left-side of the periodic table. This assessment is also supported by the wide distribution of d electrons. The broad swathe of elements in the center of periodic table are largely d-type metals with a large degree of variation in d-electron occupancy.  

What this means is that if we're looking for metals to replace, we're likely in luck because we have a large number of potential candidates to search from. If we're looking to replace semiconductors and insulators (with fully occupied s orbitals and unoccupied or partially occupied p orbitals), we have fewer choices as compared to metals.  

Since the questions framed above are targeted towards metals, we should have a fair number of candidate metals to look for in our analysis.

<a name="clean"></a>  
## Step 3. Clean and prepare data for analysis  

The VEC dataset is numerical. No non-numeric values and null values were found in the dataset. The likely reason for this is that each material in the dataset has a corresponding Composition object that can be obtained from the Materials Project API. Once one has a valid Composition object, the VEC is always likely to be returned as a numeric quantity without errors from the corresponding matminer featurizer.  

Since the first goal of this exercise is to perform a clustering analysis, we need to ensure that the variables representing s, p and d electrons are on the same scale. This is because clustering is typically (but not always) done on the basis of distance/similarity. The usual way to compute distance between two distinct datapoints is to compute their <a href = "https://en.wikipedia.org/wiki/Euclidean_distance">Euclidean Distance</a>. The side effect of this is that variables that have a larger range or smaller range than others can have an outsized influence on the clustering by making distance seem disproportionately large or small.

To overcome this, a procedure known as scaling is applied to the dataset. In essence, this procedure modifes each variable so that every variable has the same mean and standard deviation so that no variable dominates the distance computation.  

<a name="analysis"></a>
## Step 4. Data analysis, modeling and visualization

We are now in a position to perform clustering on the dataset to find the natural grouping of materials in the dataset.
As a recap - here is question 1 - When materials are represented on the basis of their average Valence Electronic Configuration (VEC), what is the optimum number of clusters that they can be grouped into? To do this, we perform clustering on the dataset using the KMeans algorithm implemented in the machine learning package <a href="https://scikit-learn.org/stable/modules/generated/sklearn.cluster.KMeans.html">scikit-learn</a>. The figure below was generated from a standard KMeans clustering analysis of the dataset for arange of cluster configurations.  

![Elbow Plot](/images/Clustering_question1.png)  

To determine the optimum number of clusters, we can use the <a href="https://en.wikipedia.org/wiki/Determining_the_number_of_clusters_in_a_data_set#The_elbow_method">Elbow Method</a>. In scikit-learn we can plot the inertia of the clustering operation (The sum of the squared distance of each point from the closest cluster) versus the number of clusters and see if an elbow exists. If it does, we can use the elbow point to determine the optimum number of clusters. If it does not we may need more anaysis.  

It is difficult to visually determine if a distinct elbow exists in the inertia plot on the left above. It is therefore useful to plot the rate of change of inertia when the number of clusters is increased. This plot on the right above shows a distinct 'knee' (continuing with anatomical metaphors) between n=3 and n=6.  

This provides a tentative answer to question 1 - It appears that the dataset based on average electronic structure can be partitioned into 3-6 distinct clusters.  

We can now answer question 2 - What materials are represented by cluster centers? Alternately, what unary material/materials are closest to cluster centers?  
Before finding out which materials are closest to the cluster centers, we first need to decide what number of clusters to use. In answering question 1, we observed that the optimum number of clusters is somewhere between 3 and 6. Picking 4 as an optimum, we can perform clustering using the same procedure as above to obtain the following tables  
  

  
| Cluster Number | Material ID |        Closest Material       | Distance to respective cluster center |
|:--------------:|:-----------:|:-----------------------------:|:-------------------------------------:|
|        1       |   mp-16960  |        AlPt<sub>2</sub>       |                  0.10                 |
|        2       |  mp-777019  | Li<sub>8</sub>SbS<sub>6</sub> |                  0.18                 |
|        3       |  mp-1183042 |       ZrSiRu<sub>2</sub>      |                  0.15                 |
|        4       |  mp-1074458 |  Mg<sub>4</sub>Si<sub>3</sub> |                  0.10                 |  



To aid intuition, we can inspect which elemental materials are closest to cluster centers. This is shown in the following table  

| Cluster Number | Material ID | Closest Unary Material | Distance to respective cluster center |
|:--------------:|:-----------:|:----------------------:|:-------------------------------------:|
|        1       |    mp-109   |           Si           |                  0.61                 |
|        2       |  mp-199937  |            K           |                  0.78                 |
|        3       |   mp-10869  |            S           |                  0.10                 |
|        4       |    mp-89    |           Cr           |                  1.11                 |

Obtaining this information enables an answer to question 3 - Does the grouping make intuitive sense - i.e. are the cluster centers sufficiently far away from each other on the basis of chemical intuition?  
<a href = "https://www.materialsproject.org/materials/mp-16960/">AlPt<sub>2</sub></a> is a stable nonmagnetic metal, <a href = "https://www.materialsproject.org/materials/mp-777019/">Li<sub>8</sub>SbS<sub>6</sub></a> is a stable ferromagnetic semiconductor, <a href = "https://www.materialsproject.org/materials/mp-1183042/">ZrSiRu<sub>2</sub></a> is a stable nonmagnetic semiconductor and  <a href="https://www.materialsproject.org/materials/mp-1074458/">Mg<sub>4</sub>Si<sub>3</sub></a> is an unstable metal.  
In case of elemental materials, Si is a semiconductor with 3 unfilled p-orbitals, K is an alkali metal with an unfilled s orbital, S is a non-metallic reactive solid with 2 unfilled p orbitals, while Cr is a transition metal with unfilled d orbitals. In terms of salient material properties, the clustering process seems to have done a reasonable job  - no material has any obvious chemical overlap.  

To aid intuition, it is useful to plot this information using a scatter plot. Since we have 3 dimensions in our dataset - s, p and d - and we wish to plot a 2D plot we can first project this data on to two dimensions using a technique called Principle Components Analysis (PCA). PCA is a technique that enables compression of information with minimum loss of information. Using PCA and plotting out data points, we see the following.  

![Cluster Plot](/images/Clustering_question3.png)

The respective cluster centers and the closest unary materials to cluster centers are also indicated in this plot. While interpreting this plot in terms of distances, it is useful to keep in mind that this is a projection of 3 dimensional data on to two dimensions and that can make some distances appear smaller or larger than they actually are.

We can move on to questions 4 and 5 that directly attempt material discovery. Let's recap question 4 - Given a target material, say Copper, what is the material (unary, binary and ternary respectively) whose VEC most closely resembles that of Copper?  
Since we have the representation of each material as a point in 3D VEC space answering this question becomes fairly simple. One merely needs to compute the distance of each point in the 3D VEC space from the point representing Copper. Doing this leads to the following table

| Closest Material | Distance to Copper |
|:----------------:|:------------------:|
|        Ag        |         0.0        |
|        Au        |         0.0        |
|       AgAu       |         0.0        |
| Ag<sub>3</sub>Au |         0.0        |
| AgAu<sub>3</sub> |         0.0        |
|       CuAu       |         0.0        |
| CuAu<sub>3</sub> |         0.0        |
|       ZnPd       |         0.0        |
|       HgPd       |         0.0        |
| Cu<sub>3</sub>Pt |        0.08        |

The first few rows involving Ag, Au are fairly intuitive. These elements fall below Copper in the same group in the periodic table. Since VEC is the same for these elements, it can be expected that materials involving combinations of these elements will be found at zero distance to Copper.  
<i>A priori</i>, the unintuitive results are those of ZnPd and HgPd. Hg and Zn are to the immediate right column while Pd is to the immediate left column of Copper in the periodic table. The situation is akin to when Gallium and Arsenic metals combine to form a semiconducting <a gref="https://en.wikipedia.org/wiki/Gallium_arsenide">GaAs</a> alloy. Gallium is to the left, while Arsenic is to the right of Silicon,which is a semiconductor. While neither material is a semiconductor by itself, semiconductivity emerges from their combination. In a similar way, ZnPd and HgPd combine to have a VEC identical to Copper. In a limited sense, we have thus performed material discovery!

We can answer question 5 - Given a target material, say Copper, what is the (binary and ternary) nitride material whose average VEC most closely resembles that of Copper? - in a similar fashion. We simply need to filter out all non-Nitride materials and then compute distances of each Nitride material from Copper. Doing this leads to the following top results. Since we know that the presence of Cu, Ag and Au will bias results to be closer to Copper, we can filter out these results as well.

| Index | Material ID | nsites | Closest Nitride Material | Distance to Copper |
|:-----:|:-----------:|:------:|:------------------------:|:------------------:|
| 33638 |  mp-542154  |   24   |          Mo3Pd2N         |      1.546541      |
| 33639 |  mp-570666  |   24   |          Mo3Pt2N         |      1.641813      |
| 18945 |   mp-10373  |    5   |          Cr3PdN          |      1.740287      |
|  7763 |  mp-1019238 |    6   |           Pd2N           |      1.852969      |
| 18946 |   mp-10374  |    5   |          Cr3PtN          |      1.912767      |
|  7759 |  mp-1080191 |    5   |           Pd3N2          |      1.922889      |
|  7768 |  mp-1189239 |   16   |           Pd3N           |      1.969609      |
| 18947 |   mp-21244  |    5   |          Cr3RhN          |      1.970176      |
| 18943 |  mp-1194250 |   28   |          Nb3Cr3N         |      2.093752      |

This  is a partial list of the top 11 Nitride materials with VEC closest to Copper not involving Cu, Ag and Au.

<a name="conclusions"></a>  
## Conclusions  
That brings us to the end of this post. Computational Materials discovery is an extremely exciting process involving several engineering tradeoffs. In addition to discovering target materials having particular properties, one needs also to ensure that these materials do not have undesirable properties. To take an example from question 5 above - A nitride material found to have good interconnect diffusion barrier properties should simultaneously exhibit low resistance. If it exhibits high resistance, it can have serious consequences on interconnect performance. We therefore need to perform the discovery process optimizing several variables simultaneously.

The end result of the materials discovery and development are extremely consequential - billions of dollars of investment in R&D, billions of chips manufactured using the new materials developed and billions of new devices like computers, phone and fitness trackers affecting billions of people over several years.

While the industrial practice of materials discovery may be far more involved than the examples provided above, my hope is that the post  serves as a blueprint for further investigation for those interested. The choice of VEC as input variables (features) for unsupervised learning and analyzing similarity was motivated by the need to keep the post accessible to a broad audience. A number of such features can be created from the materials data based on material composition and structure. A larger list of featurizers cn be accessed at the relevant <a href = "https://hackingmaterials.github.io/matminer/featurizer_summary.html">matminer table of featurizers</a>. 

Creating a good set of features for the problem at hand is often the most important step in such computational materials discovery. It is also an important part of a new sub-field in Computational Materials Science called Materials Informatics. The field of materials discovery using informatics is still nascent. Given the high stakes, it is bound to grow rapidly as data-driven techniques make their way into every industry. Go forth, find new materials with exciting properties!
