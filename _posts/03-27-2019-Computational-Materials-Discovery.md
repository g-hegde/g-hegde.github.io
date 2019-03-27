---
layout: post
title: Computational Materials Discovery - a primer
---

![Desalinating water using Graphene](https://github.com/g-hegde/g-hegde.github.io/blob/master/images/Graphene_salination.jpg)

<center><i>Desalinating water using Graphene. Image from article in <a href = "https://www.economist.com/babbage/2013/04/04/allo-allo">the Economist</a></i></center>



### <i>Materials make our world</i>.  



Does this statement seem corny and hyperbolic at first sight? I'd have to agree. While that may be true, I also hold that the statement is a self-evident truth. Still skeptical? Good. You have reasons to read on.

To make my case, let's run through a typical morning routine. You wake up, you brush your teeth using a plastic tootbrush with nylon bristles. You use a ceramic toilet. The stainless steel showerhead pours cleansing water on you. You eat breakfast while checking your phone made of a metallic alloy and coated with unbreakable Glass. Oh, and that phone contains a microprocessor chip that houses within it materials comprised of greater than 30 elements in the periodic table. You then leave for work in a car that likely has a frame strengthened with composites. You spend your day staring at an LCD/LED screen that receives display instructions from a slightly larger microprocessor chip that...is also made of materials that run the gamut of elements in the periodic table.

I won't belabor the point further. Hopefully you've caught on to my drift by now. We take these materials and the conveniences they enable for granted. This is as it should be. We shouldn't spend each moment of our waking lives worrying about how things were made. But I think it is useful to pause and reflect upon how these materials get into objects of daily use.

![Materials Continuum](https://github.com/g-hegde/g-hegde.github.io/blob/master/images/materials_continuum.jpg)
<i>Materials Continuum. Figure source: <a href = "https://www.mgi.gov/sites/default/files/documents/materials_genome_initiative-final.pdf">Materials Genome Initiative</i>

A promising material candidate is identified for an application. This step is then succeeded by several stages - development, optimization, design, certification, manufacturing and finally, deployment. The <a href = "https://www.mgi.gov/sites/default/files/documents/materials_genome_initiative-final.pdf">Materials Genome Initiative</a> estimates a timeframe of 10 to 20 years for the above process. Not to mention the fact that the discovery process itself is prone to trial-and-error and extremely time consuming. 

Advances in computation have enabled the creation of a new field - <a href="https://iopscience.iop.org/article/10.1088/1361-6463/aad926/pdf">Computational materials discovery and design</a>  - that aims to make the process above more systematic and less time-consuming. The essential idea in computational materials discovery is to reject the costly and time-consuming trial-and-error phase of discovery driven by existing chemical intuition and brute-force experimentation in favor of highly-accurate and targeted data-driven approaches.

In other words, before conducting physical experiments, one steps back, does a detailed analysis of existing materials databases or generates new data based on the fundamental physics of materials, and generates a list of promising materials suited for targeted physical properties. 

In this post, I aim to give the reader a brief but hopefully informative snapshot of the computational materials discovery process. In the interest of simplicity, I use the <a href='https://en.wikipedia.org/wiki/Electron_configuration'>Electronic Configuration</a> of atoms to group materials and find materials having properties similar to industrially relevant materials.

At the outset, I caution that the post is not intended to be exhaustive conceptually. Instead, I hope to convey a sense for how the problem of materials discovery can be tackled using a principled, data-based approach using computational tools that are free and easily accessible.

The rest of this post is organized as follows. First, I attempt to provide an intuitive explanation of the concept of average electronic configuration of materials. I then describe the use of two Python software packages - <a href = "http://http://pymatgen.org/">pymatgen</a> and <a href = "https://hackingmaterials.github.io/matminer/">matminer</a> - in creating a database of the valence electron configuration of a large number of inorganic materials. I then outline specific questions in the context of materials similarity and discovery that can be answered by performing a detailed analysis of database. Each subsequent section provides answers to these questions in the form of visualizations and tables. The last section provides key take-home messages from the analysis and some suggestions for expanding and improving upon the analysis in this post. Accompanying code for this post can be found at my repository on <a href = "https://github.com/g-hegde/material-similarity">GitHub</a>. 
