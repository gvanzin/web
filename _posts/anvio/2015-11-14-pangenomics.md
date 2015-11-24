---
layout: post
title: "An anvi'o workflow for microbial pangenomics"
excerpt: "The user-friendly interface anvi'o provides to work with pangenomes."
modified: 2015-10-14
tags: []
categories: [anvio]
comments: true
authors: [tom, meren]
---

{% include _toc.html %}


That's why we are happy to demonstrate how [anvi'o](https://peerj.com/articles/1319/) can be used to process, visualize, and manipulate pangenomic data in its user-friendly environment. Although we are still [working](http://github.com/meren/anvio) on new modules for a fully automatized pangenomic workflow, the current anvi'o interface already provide its user with the opportunity to analyze pangenomes combined with a variety of contextual data, and to generate high-quality, publication-ready figures.

Our goal in this article is to describe original pangenomic analyses of publically available genomic data to demonstrate the anvi'o pangenomic workflow. Please don't hesitate to use the comments section for your suggestions, our [discussion group]({% post_url anvio/2015-10-14-anvio-discussion-group %}) for your technical questions. You can also keep an eye on our [public code repository](http://github.com/meren/anvio) for new releases, or to report issues.

Please note that you can find every file mentioned in this article, including run scripts to invoke anvi'o interactive interface here: [http://dx.doi.org/10.6084/m9.figshare.1603512](http://dx.doi.org/10.6084/m9.figshare.1603512).


You will find all files metnioned in this section in `01-ECOLI-PANGENOME/` directory of our figshare archive.

### Generating protein clusters for the data matrix

{:.notice}
If you would like to repeat this analysis on your collection of genomes, you can use any gene finder that reports proper genbank files for open reading frames identified, and use those genbank files instead (we thought prodigal would work, but the genbank files prodigal generates does not seem to be compatible with genbank file format ITEP expects. Why? Because <a href="https://twitter.com/merenbey/status/662373313826693120">reasons</a>).

We clustered amino acid sequences for 47,415 open reading frames in these genomes using [ITEP](https://price.systemsbiology.org/research/itep/) ("Integrated Toolkit for Exploration of Pan-genomes", a useful tool we were introduced by our colleague [Rika Anderson](https://twitter.com/RikaEAnderson)). ITEP identified a total of 7,795 protein clusters in our collection, where each cluster contained 1 to 102 proteins. 

The main purpose of this step is to create a basic matrix to connect genomes and protein clusters. Although we used ITEP, any other solution to cluster protein sequences could have been used to acquire this essential input file.

We stored this information as a matrix of protein clusters and their occurrences across 10 genomes (see `data.txt` in our figshare archive) which should look like the following table, where each cell shows the count of a given protein cluster (first column) in each genome (first row):

|contig|O157_H7-EDL933|CFT073|K-12_W3110|O157-H7-Sakai|SE11|SE15|BL21-DE3|K-12_MG1655|O103-H2-12009|O111-H-11128|
|:--|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
|1|18|8|0|18|0|0|0|0|23|35|
|2|19|9|0|18|0|0|0|0|23|33|
|3|2|1|6|1|1|4|56|6|1|1|
|4|10|2|2|11|2|1|3|1|10|12|
|5|9|3|2|11|2|1|1|2|10|10|
|6|0|0|7|1|1|3|29|7|1|2|
|7|5|6|4|4|7|5|3|4|6|5|
|8|14|6|0|13|0|5|0|0|6|3|
|9|9|3|0|10|4|1|1|0|9|9|
|(...)|(...)|(...)|(...)|(...)|(...)|(...)|(...)|(...)|(...)|(...)|
|7795|0|0|0|0|0|0|0|0|0|1|


When you have this script, you can run this command to get the `data.txt` (if you didn't have the script and ended up downloading it, put "python" at the beginning of the following command):

{% highlight bash %}
Num genomes ..................................: 10
Num protein clusters .........................: 7,795
Data file is generated .......................: data.txt



{% highlight bash %}

Note the `--transopse` flag. The resulting tree will describe the clustering of genomes with respect to protein clusters they harbor. We stored the resulting tree in `samples-order.txt` file, the format of which is explained [here](http://merenlab.org/2015/11/10/samples-db/#the-samples-order-file-format).

We also have another file that provides us with more information about our genomes. This file is called `samples-information.txt`, and it looks like this:

|samples|Nb_proteins|GC-content|
|:----|:----:|:----:|
|BL21-DE3|4192|50.8|
|CFT073|5364|50.5|
|O103-H2-12009|5049|50.7|
|O111-H-11128|4967|50.6|
|O157_H7-EDL933|5285|50.4|
|O157-H7-Sakai|5204|50.5|
|SE11|4673|50.8|
|SE15|4336|50.7|
|K-12_MG1655|4132|50.8|
|K-12_W3110|4213|50.8|

Using these two files, we then generate the samples database:

{% highlight bash %}

Please read [this blog post](http://merenlab.org/2015/11/10/samples-db/) to better make sense of anvi'o samples databases.
We are almost done!

We are going to create a mock FASTA file that contains entries for each protein to keep the interactive interface happy. For this, we simply take every protein cluster ID from our `data.txt` matrix, and add into a FASTA file with some mock sequences (clearly you can put in the actual sequences in this file, but the current version of anvi'o will not do much with them):

{% highlight bash %}
for i in `awk '{if(NR > 1) print $1}' data.txt`; do echo ">$i"; echo "ATCG"; done > fasta.fa
{% endhighlight %}
<a href="{{ site.url }}/images/anvio/2015-11-14-pan-genomics/pan-genome-1.png"><img src="{{ site.url }}/images/anvio/2015-11-14-pan-genomics/pan-genome-1.png" style="border: None;" /></a>
</div>

<div class="quotable">
Figure 1: Anvi'o pangenome display of ten *Escherichia coli* genomes with a total of 7,795 protein clusters. The inner tree (from `tree.newick`) organizes protein clusters based on their occurrence patterns across genomes. Layers around the tree display the presence/absence of protein clusters in a given genome (from `data.txt`). Note that the genome layers are organized based on the distribution of protein clusters (from `SAMPLES.db`). Finally, the number of proteins and GC-content (also from `SAMPLES.db`) is displayed to demonstrate the capabilities of anvi'o incorporating contextual data as a natural extention of the pangenome. 

<iframe width="560" height="315" src="https://www.youtube.com/embed/RPjdl0bFCn0" frameborder="0" allowfullscreen></iframe>
<div style="padding-bottom: 50px;"></div>


Using ITEP, we organized the 119,511 genes identified in these genomes into 14,981 protein clusters, and generated all the relevant files as described in the first section. As an additional step, this time we also uploaded genomes into [RAST](http://rast.nmpdr.org/) to recover their metabolic profiles, and incorporated this contextual information into the metadata table before generating the `SAMPLES.db` file. All relevant files for this analysis are available in the directory `02-ECOLI-SAUREUS-SENTERICA-PANGENOME` in our figshare archive.
<a href="{{ site.url }}/images/anvio/2015-11-14-pan-genomics/pan-genome-2.png"><img src="{{ site.url }}/images/anvio/2015-11-14-pan-genomics/pan-genome-2.png" style="border: None;" /></a>
</div>
</div>

## Quick workflow for experimented users

The following script describes an overview of steps detailed in this post:

<script src="https://gist.github.com/meren/e99c46484ca5a479cf0c.js"></script>

Please don't hesitate to use the comments section for your suggestions, our [discussion group]({% post_url anvio/2015-10-14-anvio-discussion-group %}) for your technical questions. You can keep an eye on our [public code repository](http://github.com/meren/anvio) for new releases.