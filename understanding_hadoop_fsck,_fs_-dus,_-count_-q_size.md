# Understanding hadoop fsck, fs -dus, -count -q size output

Many hadoop users often confuse size and significance hadoop fsck, hadoop fs -dus, hadoop -count -q and other hadoop file system command output. 
Here to do a summary of these issues. First we have to clear two concepts:

* Logical space, a distributed file system that is on the real file size
* Physical space, a distributed file system that is present on the actual space occupied by the file

###Why logical space is generally not equal to the physical space? 

In order to ensure the reliability of distributed file system files, often to save multiple backups (usually 3 parts), under long backups is not 1, the general logic of the physical space would be several times the space. Relationship is as follows:

>HDFS physical space = logical space * block backups

`hadoop fsck and hadoop fs -dus`

execution hadoop fsck and hadoop fs -dus display file size represents the logical files take up space.

```
$ Hadoop fsck / path / to / Directory
  Total size :     16,565,944,775,310 B     <===  look here 
 Total dirs :     3922 
 Total Files :    418 464 
 Total Blocks ( validated ):       502705  ( avg . Block size 32,953,610 B ) 
 Minimally replicated Blocks :    502 705  ( 100.0  %) 
 Over - replicated Blocks :         0  ( 0.0  %) 
 Under - replicated Blocks :        0  ( 0.0  %) 
 Mis - replicated Blocks :          0  ( 0.0  %) 
 Default replication factor :     3 
 Average Block replication :      3.0 
 Corrupt Blocks :                 0 
 Missing Replicas :               0  ( 0.0  %) 
 Number of Data - nodes :           18 
 Number of racks :                1 
FSCK ended at Thu  Oct  20  20 : 49 : 59 CET 2011  in  7516 milliseconds
 
The filesystem path under '/ path / to / Directory'  is HEALTHY

$ Hadoop FS - DUS / path / to / Directory
hdfs : // master: 54310 / path / to / Directory 16,565,944,775,310 <=== see here
```