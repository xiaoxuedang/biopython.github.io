---
title: SeqIO
permalink: wiki/SeqIO
layout: wiki
---

The page describes Bio.SeqIO, a new Sequence Input/Output interface for
BioPython.

Some code has now been checked into CVS, and other bits are available on
[Bug 2059](http://bugzilla.open-bio.org/show_bug.cgi?id=2059). Details
are currently being discussed on the [Development mailing
list](http://biopython.org/wiki/Mailing_lists).

If all goes well, the code will be available in the next release,
probably BioPython 1.43.

Aims
----

We would like to recreate the simplicity of [BioPerl's
SeqIO](http://www.bioperl.org/wiki/HOWTO:SeqIO), and in the long term
its [impressive list of supported file
formats](http://www.bioperl.org/wiki/Sequence_formats).

As currently implemented, the BioPython code covers multiple alignment
file formats as well. Alignment specific handling may be required in the
future should the BioPython alignment object become capable of holding
more than just sequence level annotation. See also BioPerl's list of
[multiple alignment
formats](http://www.bioperl.org/wiki/Multiple_alignment_formats).

Bio.SeqIO provides a simple uniform interface to assorted file formats,
but will *only* return sequences as SeqRecord objects.

In some cases, this does lead to some apparent duplication. For example,
Bio.SeqIO and Bio.Nexus will both read sequences from Nexus files.
However, Bio.Nexus can also do much more, for example reading any
phylogenetic trees in a Nexus file.

Peter

File Formats
------------

At the moment (unlike BioPerl::SeqIO) multiple alignment file formats
are treated much like any the other sequence file, but with the
constraint that all the (gapped) sequences must be the same length.

| Format name | Reads | Writes | Notes                                     |
|-------------|-------|--------|-------------------------------------------|
| fasta       | Yes   | Yes    |                                           |
| genbank     | Yes   | No     | Uses Bio.GenBank                          |
| clustal     | Yes   | No     |                                           |
| nexus       | Yes   | No     | Also known as PUAP format. Uses Bio.Nexus |
| phylip      | Yes   | Yes    | Truncates names at 10 characters.         |
| stockholm   | Yes   | Yes    | Also known as PFAM format.                |
||

Helper Functions
----------------

There are four helper functions which all take a filename, and optional
format. Each sequence is returned as a SeqRecord object.

-   File2SequenceIterator, returns a iterator (low memory, forward
    access only)
-   File2SequenceList, returns a list of sequences (high memory,
    random access)
-   File2SequenceDict, returns a dictionary of sequences (high memory,
    random access by ID)
-   File2Alignment, returns an alignment object (for use with multiple
    sequence alignment file formats)

For sequential file formats (like Fasta, GenBank, EMBL etc) the file can
be read record by record, and thus the iterator interface can save a
significant amount of memory (RAM) which allows you to deal with very
large files.

For interlaced file formats (like Clustal/Clustalw or annotated
Stockholm files) the entire file must be read in one go. You may not
save much memory by using the SequenceIterator in this case, but it is
provided so that you shouldn't need to re-write your code if you change
the input file format.

For writing records to a file there is a single helper function:

-   Sequences2File, takes records and filename

For sequential files, it may be possible to write the records to file
one by one. You will have to use the appropriate format writer object
directly for that.

Examples using the Helper Functions
-----------------------------------

Suppose you have a fasta file, example.fasta, which you wish to load:

`from Bio.SeqIO import File2SequenceIterator`  
`for record in File2SequenceIterator("example.fasta") :`  
`    print record.id`  
`    print record.seq`

In this example, BioPython has guessed the file format from the
extension.

Adding new file formats
-----------------------

To add support for reading a new file format, you just have to implement
an iterator that expects a just file handle and returns SeqRecord
objects. You may do this using:

-   A generator function (using the yield keyword)
-   Your own iterator class (consider subclassing something from
    Bio.SeqIO.Interfaces for this)
-   Build a list of SeqRecords and then turn it into a list iterator
    using the iter() function.

You may accept additional *optional* arguments (alphabet for example).

The the new format must be added to the relevant mappings in
Bio/SeqIO/\_\_iter\_\_.py plus any standard file extensions.