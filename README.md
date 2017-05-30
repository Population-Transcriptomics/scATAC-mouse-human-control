C1 ATAC-seq cross-contamination control
=======================================

_Mouse / human control of cross-contaminations in single-cell ATAC-seq on Fluidigm C1._

The purpose of this repository is to announce, document, and track issues
and contributions related to a single-cell ATAC-seq dataset that we created
to measure and control cross-contaminations.  Please feedback with the issue
tracker here on GitHub, or with the [Biostars](https://www.biostars.org/p/236583/)
forum.

To rule out or quantify the amount of cross-contamination in single-cell
ATAC-seq libraries prepared on the C1 platform with the protocol available
on [ScriptHub][], we performed the control experiemnts described below,
loading a mixture of mouse and human cells.

[ScriptHub]: https://www.fluidigm.com/c1openapp/scripthub/script/2015-06/single-cell-chromatin-accessib-1433443631246-1

We used medium-sized IFCs that had the original design prone to [doublet
capture][].  The chambers were imaged at multiple focal planes and the
images are available in our [single-cell data integration platform][].  The
sequencing data is available on Zenodo, at the links displayed below for
each experiment.

[doublet capture]: http://info.fluidigm.com/FY16Q2-C1WhitePaperUpdate_LP.html
[single-cell data integration platform]: http://single-cell.clst.riken.jp/ATAC_human_mouse_control_metadata_list.php

Experiment 1: mouse / human cell mixture
----------------------------------------

In the first experiment, we loaded mouse and human cells together, after
staining them with different calceins.

 - Hep G2 (human), stained witn green calcein. 12.8 μm diameter in average.
 - Hepa 1-6 (mouse), stained wiht red calcein. 13.3 μm diameter in average.
 - IFC ID 1772-123-148 (old design)
 - MiSeq run ID: 161026_M00528_0239_000000000-ANY8K
   <https://doi.org/10.5281/zenodo.263694>

This experiment gives an estimate of the cross-contaminations in situations
where the cells are robust and the possibility of damage before loading was
limited.

 
Experiment 2: FACS-sorted mouse and human cells with / without washing
----------------------------------------------------------------------

In the second experiment, we FACS-sorted the cells before loading, thus adding
one potential source of stress and damage.  Nevertheless, we used gentle FACS
conditions (nozzle size of 100 μm), to reflect the actual settings that would be
recommended when working with fragile cells.  We loaded the cells with or without
washing them after FACS-sorting.  48 samples (single cells, empty chambers,
doublets) from each C1 run were multiplexed in one MiSeq run.

Cell preparation:

 - 2~5,000,000 cells collected in PBS and then FACS-sorted.

Brief experimental summary:

 - Same cell types and staining as in experiment 1.
 - sorting (collection into native medium)
 - cell count -> load in first C1 machine (IFC ID 1772-123-155, old design)
 - 300 × _g_, 5 min, then discard supernatant
 - resuspend with Cell Wash buffer (for C1)
 - cell count -> load in second C1 machine (IFC ID 1772-123-158 old design)
 - 250 cells/ul preparation (10,000 - 30,000 cells needed)
 - MiSeq run ID: 170116_M00528_0252_000000000-B3BKB
   <https://doi.org/10.5281/zenodo.263695>


Microscope pictures of the capture chambers
-------------------------------------------

 - Overview summary:  
   http://single-cell.clst.riken.jp/ATAC_human_mouse_control_summary_view.php 

 - JPEG and stack movies:
   http://single-cell.clst.riken.jp/ATAC_human_mouse_control_metadata_list.php 

 - Bulk download JPEGs:
   http://single-cell.clst.riken.jp/files/ATAC-human-mouse-control/All_runs_JPEG_cell_images/ 
       
       f82894834070834225ab235be65514e4  1772-123-148.tar.gz
       8eaebae1ed41622d669b7832188aa39a  1772-123-155.tar.gz
       12fe7663bda3afb5d4b1f8fcdfd0a222  1772-123-158.tar.gz

 - Bulk download MPGs:
   http://single-cell.clst.riken.jp/files/ATAC-human-mouse-control/mpg_files/mpg_bulkdownload/ 
       
       b55fd18aede5d53ff9758dfcf7095378  1772-123-148_mpg.tar.gz
       5f532b9c2a59da5388a55155dbc6ddbe  1772-123-155_mpg.tar.gz
       1ced7ef23cae818f659986c76f0a19d6  1772-123-158_mpg.tar.gz



Data analysis
-------------

Our data analysis is ongoing.  Here is a description of our [alignment
strategy](sequenceAlignment.md).  Comments and suggestions are welcome (ideally
via [Biostars](https://www.biostars.org/)).
