Quickstart
==========

Quick importation
-----------------
pyGeno's database is populated by importing data wraps.
pyGeno comes with a few datawraps, to get the list you can use:

.. code:: python
	
	import pyGeno.bootstrap as B
	B.printDatawraps()

.. code::

	Available datawraps for bootstraping
	
	SNPs
	~~~~|
	    |~~~:> Homo_sapiens_agnostic.dummySRY.tar.gz
	    |~~~:> Homo_sapiens.dummySRY_casava.tar.gz
	    |~~~:> dbSNP142_human_GRCh37_common_all.tar.gz
	    |~~~:> dbSNP142_human_common_all.tar.gz
	
	
	Genomes
	~~~~~~~|
	       |~~~:> Homo_sapiens.GRCh37.75.tar.gz
	       |~~~:> Homo_sapiens.GRCh37.75_Y-Only.tar.gz
	       |~~~:> Homo_sapiens.GRCh38.78.tar.gz
	       |~~~:> Mus_musculus.GRCm38.78.tar.gz

To get a list of remote datawraps that pyGeno can download for you, do:

.. code:: python

	B.printRemoteDatawraps()


Importing whole genomes is a demanding process that take more than an hour and requires (according to tests) 
at least 3GB of memory. Depending on your configuration, more might be required.

That being said importating a data wrap is a one time operation and once the importation is complete the datawrap
can be discarded without consequences.

The bootstrap module also has some handy functions for importing built-in packages.

Some of them just for playing around with pyGeno (**Fast importation** and **Small memory requirements**):

.. code:: python
	
	import pyGeno.bootstrap as B

	#Imports only the Y chromosome from the human reference genome GRCh37.75
	#Very fast, requires even less memory. No download required.
	B.importGenome("Homo_sapiens.GRCh37.75_Y-Only.tar.gz")
	
	#A dummy datawrap for humans SNPs and Indels in pyGeno's AgnosticSNP  format. 
	# This one has one SNP at the begining of the gene SRY
	B.importSNPs("Homo_sapiens.dummySRY_casava.tar.gz")

And for more serious work, the whole reference genome.

.. code:: python

	#Downloads the whole genome (205MB, sequences + annotations), may take an hour or more.
	B.importGenome("Homo_sapiens.GRCh38.78.tar.gz")

That's it, you can now print the sequences of all the proteins that a gene can produce::

	from pyGeno.Genome import Genome
	from pyGeno.Gene import Gene
	from pyGeno.Protein import Protein

	#the name of the genome is defined inside the package's manifest.ini file
	ref = Genome(name = 'GRCh37.75')
	#get returns a list of elements
	gene = ref.get(Gene, name = 'SRY')[0]
	for prot in gene.get(Protein) :
		  print prot.sequence

You can see pyGeno achitecture as a graph where everything is connected to everything. For instance you can do things such as::

	gene = aProt.gene
	trans = aProt.transcript
	prot = anExon.protein
	genome = anExon.genome

Queries
-------

PyGeno allows for several kinds of queries::

	#in this case both queries will yield the same result
	myGene.get(Protein, id = "ENSID...")
	myGenome.get(Protein, id = "ENSID...")
	
	#even complex stuff
	exons = myChromosome.get(Exons, {'start >=' : x1, 'stop <' : x2})
	hlaGenes = myGenome.get(Gene, {'name like' : 'HLA'})

	sry = myGenome.get(Transcript, { "gene.name" : 'SRY' })

To know the available fields for queries, there's a "help()" function::

	Gene.help()


Faster queries
---------------

To speed up loops use iterGet()::
	
	for prot in gene.iterGet(Protein) :
	  print prot.sequence

For more speed create indexes on the fields you need the most::
	
	Gene.ensureGlobalIndex('name')


Personalized Genomes
--------------------

Personalized Genomes are a powerful feature that allow to work on the specific genomes and proteomes of your patients. You can even mix several SNPs together::
	
	from pyGeno.Genome import Genome
	#the name of the snp set is defined inside the datawraps's manifest.ini file
	dummy = Genome(name = 'GRCh37.75', SNPs = 'dummySRY')
	#you can also define a filter (ex: a quality filter) for the SNPs
	dummy = Genome(name = 'GRCh37.75', SNPs = 'dummySRY', SNPFilter = myFilter())
	#and even mix several snp sets
	dummy = Genome(name = 'GRCh37.75', SNPs = ['dummySRY', 'anotherSet'], SNPFilter = myFilter())

pyGeno allows you to customize the Polymorphisms that end up into the final sequences. It supports SNPs, Inserts and Deletions::
	
	from pyGeno.SNPFiltering import SNPFilter
	from pyGeno.SNPFiltering import SequenceSNP

	class QMax_gt_filter(SNPFilter) :

		def __init__(self, threshold) :
			self.threshold = threshold

		def filter(self, chromosome, dummySRY = None) :
			if dummySRY.Qmax_gt > self.threshold :
				#other possibilities of return are SequenceInsert(<bases>), SequenceDelete(<length>)
				return SequenceSNP(dummySRY.alt)
			return None #None means keep the reference allele

	persGenome = Genome(name = 'GRCh37.75_Y-Only', SNPs = 'dummySRY', SNPFilter = QMax_gt_filter(10))

