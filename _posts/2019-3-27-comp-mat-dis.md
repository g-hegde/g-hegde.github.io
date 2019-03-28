---
layout: post
title: Computational Materials Discovery - a primer
---

![Graphene Salination](/images/Graphene_salination.jpg)

<i>Desalinating water using Graphene. Image from article in <a href = "https://www.economist.com/babbage/2013/04/04/allo-allo">the Economist</a></i>



### <i>Materials make our world</i>.  



Does this statement seem corny and hyperbolic at first sight? I'd have to agree. While that may be true, I also hold that the statement is a self-evident truth. Still skeptical? Good. You have reasons to read on.

To make my case, let's run through a typical morning routine. You wake up, you brush your teeth using a plastic tootbrush with nylon bristles. You use a ceramic toilet. The stainless steel showerhead pours cleansing water on you. You eat breakfast while checking your phone made of a metallic alloy and coated with unbreakable Glass. Oh, and that phone contains a microprocessor chip that houses within it materials comprised of greater than 30 elements in the periodic table. You then leave for work in a car that likely has a frame strengthened with composites. You spend your day staring at an LCD/LED screen that receives display instructions from a slightly larger microprocessor chip that...is also made of materials that run the gamut of elements in the periodic table.

I won't belabor the point further. Hopefully you've caught on to my drift by now. We take these materials and the conveniences they enable for granted. This is as it should be. We shouldn't spend each moment of our waking lives worrying about how things were made. But I think it is useful to pause and reflect upon how these materials get into objects of daily use.

![Materials Continuum](/images/materials_continuum.jpg)
<i>Materials Continuum. Figure source: <a href = "https://www.mgi.gov/sites/default/files/documents/materials_genome_initiative-final.pdf">Materials Genome Initiative</a></i>

A promising material candidate is identified for an application. This step is then succeeded by several stages - development, optimization, design, certification, manufacturing and finally, deployment. The <a href = "https://www.mgi.gov/sites/default/files/documents/materials_genome_initiative-final.pdf">Materials Genome Initiative</a> estimates a timeframe of 10 to 20 years for the above process. Not to mention the fact that the discovery process itself is prone to trial-and-error and extremely time consuming. 

Advances in computation have enabled the creation of a new field - <a href="https://iopscience.iop.org/article/10.1088/1361-6463/aad926/pdf">Computational materials discovery and design</a>  - that aims to make the process above more systematic and less time-consuming. The essential idea in computational materials discovery is to reject the costly and time-consuming trial-and-error phase of discovery driven by existing chemical intuition and brute-force experimentation in favor of highly-accurate and targeted data-driven approaches.

In other words, before conducting physical experiments, one steps back, does a detailed analysis of existing materials databases or generates new data based on the fundamental physics of materials, and generates a list of promising materials suited for targeted physical properties. 

In this post, I aim to give the reader a brief but hopefully informative snapshot of the computational materials discovery process. In the interest of simplicity, I use the <a href='https://en.wikipedia.org/wiki/Electron_configuration'>Electronic Configuration</a> of atoms to group materials and find materials having properties similar to industrially relevant materials.

At the outset, I caution that the post is not intended to be exhaustive conceptually. Instead, I hope to convey a sense for how the problem of materials discovery can be tackled using a principled, data-based approach using computational tools that are free and easily accessible.

The rest of this post is organized as follows. First, I attempt to provide an intuitive explanation of the concept of average electronic configuration of materials. I then describe the use of two Python software packages - <a href = "http://http://pymatgen.org/">pymatgen</a> and <a href = "https://hackingmaterials.github.io/matminer/">matminer</a> - in creating a database of the valence electron configuration of a large number of inorganic materials. I then outline specific questions in the context of materials similarity and discovery that can be answered by performing a detailed analysis of database. Each subsequent section provides answers to these questions in the form of visualizations and tables. The last section provides key take-home messages from the analysis and some suggestions for expanding and improving upon the analysis in this post. Accompanying code for this post can be found at my repository on <a href = "https://github.com/g-hegde/material-similarity">GitHub</a>. 

### Electronic configuration

The <a href='https://en.wikipedia.org/wiki/Electron_configuration'>Electronic Configuration</a> of atoms describes how electrons are distributed in an atom. In a broad sense, atoms are comprised of

* A central core called the nucleus, which contains neutrons and positively-charged protons  
* Negatively-charged electrons that orbit the nucleus

Electrons orbiting the nucleus do so in spatial configurations called orbitals at multiple energy levels (shells). Orbitals come essentially in 5 different flavors - s, p, d, f and g. Each orbital can hold a maximum of 2 electrons. At each energy level, one has a combination of s and/or p, d, f and g orbitals. s-orbitals are spherically symmetric and there is consequently only one s orbital for each energy level. There are 3 and 5 p and d orbitals respectively for energy levels in which these orbitals exist. Valence electrons are electrons that occupy the outermost energy levels. These are less-bound to the nucleus and are consequently freer to move through the body of the material. Valence electrons and their resulting configuration is what determines bonding and chemical properties of a material. The picture below shows the Valence Electronic Configuration (VEC) of elements in the periodic table.

![Periodic_Table.jpg](/images/Periodic_Table.jpg)  
<i>Image from <a href = "https://opentextbc.ca/chemistry/chapter/6-4-electronic-structure-of-atoms-electron-configurations/">OpenBC Textbook</a></i>

The VEC of materials is one of their most fundamental physical properties. Knowledge of the VEC of atoms in materials can help us understand and predict a wide range of their physical and chemical properties. For instance, knowledge of the VEC of Silicon *3s<sup>2</sup>3p<sup>2</sup>* explains why it is a semiconductor. Or why Copper, Silver and Gold *ns<sup>1</sup>(n-1)d<sup>10</sup>* are metals.

![Si_VEC.png](/images/Si_VEC.png) 
<i><a href = "http://www.chem4kids.com/files/elements/014_shells.html">Silicon Electronic shell levels</a>. The Valence Electronic Configuration of Silicon is*3s<sup>2</sup>3p<sup>2</sup>*. The outermost shell (3) contains a filled s orbital and two partially filled p orbitals</i>  

Developing a deep understanding of electronic configuration requires extensive knowledge of atomic physics and chemistry. This is because, at a fundamental level, the behavior of all materials is governed by the rules of <a href="https://en.wikipedia.org/wiki/Quantum_mechanics">Quantum Mechanics</a>. Needless to say, this is beyond the scope of a single blog post. It is useful instead to focus on understanding how the VEC changes from one atom/material to another.

The periodic table above shows the VEC of all elemental materials (The terms elemental/unary/mono-atomic will be used interchangeably here). For compound materials such as binary materials (materials comprised of two elements) and ternary materials (three elements), such a straightforward comparison is not possible.


### Average electronic configuration

For materials containing multiple elements, a simple tweak to valence electronic structure is obtained as follows. An <a href = "http://chemed.chem.purdue.edu/genchem/topicreview/bp/ch13/unitcell.php">Unit Cell</a> of a material is a repeatable unit of atoms of a material. When this unit is repeated uniformly in all spatial dimensions, we obtain a crystal of the material. By summing the VEC of each atom in the Unit Cell and dividing through by the number of atoms in the unit cell we obtain the average VEC.

Having seen what the average VEC is, we can put this knowledge to use in a simple materials discovery case study.

### Materials-Discovery case-study  

We have a material that we seek to replace with another cheaper, equally performing alternative. As a Computational Materials Discovery scientist, it is your responsibility to create a database with relevant VEC information for all possible elements and compounds that may be suitable replacements for this material. You are then required to clean this database to ensure the fidelity of all the data in the database. After this, you are required to perform unsupervised machine learning to see which materials group together. The set of materials closest to the target material can then be pushed up the chain for further consideration as replacement candidates.

### Relevant questions  
* When materials are represented on the basis of their average Valence Electronic Configuration (VEC), what is the optimum number of clusters that they can be grouped into?  
* What materials are represented by cluster centers? Alternately, what unary material/materials are closest to cluster centers? 
* Does the grouping make intuitive sense - i.e. are the cluster centers sufficiently far away from each other on the basis of chemical intuition?  
* Given a target material, say Copper, what is the material (unary, binary and ternary respectively) whose VEC most closely resembles that of Copper? Performing such an analysis could be useful for finding replacements for <a href = "https://en.wikipedia.org/wiki/Copper_interconnects"> Copper interconnects</a> in microprocessors.
* Given a target material, say Copper, what is the (binary and ternary) nitride material whose average VEC most closely resembles that of Copper?  Performing such an analysis could be useful for finding suitable diffusion barrier materials for Copper interconnects.

