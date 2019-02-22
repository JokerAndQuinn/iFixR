

# Mimic

* [I. Introduction of Mimic](#user-content-i-introduction)
* [II. Environment setup](#user-content-ii-environment)
* [III. Run SimFix Step-by-Step](#user-content-iii-how-to-run)
* [IV. Evaluation Result](#user-content-iv-evaluation-result)
* [V. Generated Patches](#user-content-v-generated-pacthes)
* [VI. Structure of the project](#user-content-vi-structure-of-the-project)

## I. Introduction

*Mimic* a patch generation system for user-reported bugs

#is an automatic program repair technique, which leverages exisiting patches from other projects and similar code snippets in #the same project to generate patches. The following figure is the workflow of our approach.

![The workflow of this technique.\label{workflow}](docs/figures.png)

#### Mining Stage

1. mine repair patterns from existing open-source projects, after which we can obtain a set of frequent repair patterns. Those patterns can be reused for other repairing scenarios as well.

#### Repairing Stage

1. **Fault Localization** : obtain a ranking list of candidate faulty statements and extract corresponding code snippets.
2. **Donor Snippet Identification** : identify the similarity between faulty code snippet and each candidate similar code snippet by leveraging three similarity metrics, according to which we obtain a list of candidate similar snippets with decending order of similarity value.
3. **Variable Mapping** : establish the mapping relationship between variables in faulty and similar code snippets by leveraging similarity metrics and then obtain a mapping table, based on which the variables in the donor code snippet will be replaced with the corresponding variables.
4. **Modification Extraction and Intersection** : extract code modifications to faulty code snippet via AST (Abstract Syntax Tree) matching and differencing against the donor snippet, and then the frequent patterns from the mining stage will be used to take intersection with those modifications to further ruled out invalid ones.
5. **Patch Generation & Validation** : generate repair patches by applying extracted code modifications to the faulty code snippet with combining and ranking whose modifications, then using the test suite to validate the correctness of candidate patches until a correct patch found or timeout.


## II. Environment

* OS: macOS Mojave (10.14.3)
* JDK7: Oracle jdk1.7 (**important!**)
* JDK8: Oracle jdk1.8 (**important!**)
* Download and configure Anaconda
* Create an python environment using the [environment file](environment.yml)
  ```powershell
  conda env create -f environment.yml
  ```




## III. How to run

* Active the conda environment from shell
  ```powershell
  source activate python36
  ```


#### Before running

* Update [config file](config.yml) with corresponding user paths.

#### Running Options 

Our prototype of *SimFix* needs **three** input options for running.

* `--proj_home ` : the home of buggy program of benchmark. (`${buggy_program_path}` for the example)

* `--proj_name` : the project name of buggy program of benchmark. (`chart` for the example)

* `--bug_id` : the identifier of the buggy program. (`1` for the example)

  * the option of `--bugy_id` supports multiple formats:

    `single_id` : repair single bug, `e.g., 1`.

    `startId-endId` : repair a series of bugs with consecutive identifiers, `e.g., 1-3`.

    `single_id,single_id,single_id` : repair any bugs for the specific program, `e.g., 1,5,9`.

    `all` : repair all buggy versions of a specific project, `i.e., all`.

  ```powershell
  Usage: --proj_home=${proj_home} --proj_name=${proj_name} --bug_id=${bug_id}
  Example: --proj_home=/home/user --proj_name=chart --bug_id=1
  Another: --proj_home=/home/user --proj_name=chart --bug_id=1,4,8
  ```

**OPTION 1** : run within eclipse (please use the old version: tested on **[Mars](https://www.eclipse.org/mars/)**, which depends on Java7).

* From the Main class:

   `Run As`→`Run Configurations…` →`Arguments` : set the above arguments as *Program Arguments*.

**OPTION 2** : run using command line.

* We also provide runnable jar file of *SimFix* in the home folder of the project `i.e., simfix.jar`.

  set the home directory of the *SimFix* project as your correct path and then run as:

  `java -jar simfix.jar --proj_home=/home/user --proj_name=chart --bug_id=1`

#### Step 3, Result Analysis

After finishing the repair, there will be two additional folders: `log` and `patch`.

* `log` : debug output, including buggy statements already tried, patches and reference code snippet for correct patch generation.

* `patch` : a single source file repaired by *SimFix* that can pass the test suite. In the source file, you can find the patch, which is formatted as (example of Chart_1):

  ```java
  // start of generated patch
  int index=this.plot.getIndexOf(this);
  CategoryDataset dataset=this.plot.getDataset(index);
  if(dataset==null){
  return result;
  }
  // end of generated patch
  /* start of original code
          int index = this.plot.getIndexOf(this);
          CategoryDataset dataset = this.plot.getDataset(index);
          if (dataset != null) {
              return result;
          }
   end of original code*/
  ```

## IV. Evaluation Result

<!-- ![Result Comparison.\label{workflow}](docs/test) -->
APR tool | Lang | Math | Total
-------- | ---- | ---- | -----
jGenProg  | 0/0 | 5/18 | 5/18  
jKali  | 0/0 | 1/14 | 1/14  
jMutRepair  | 0/1 | 2/11 | 2/12  
HDRepair  | 2/6 | 4/7 | 6/13  
Nopol  | 3/7 | 1/21 | 4/28  
ACS  | 3/4 | 12/16| 15/20  
ELIXIR  | 8/12| 12/19| 20/31  
JAID  | 1/8 | 1/8 | 2/16  
ssFix  | 5/12| 10/26| 15/38  
CapGen  | 5/5 | 12/16| 17/21  
SketchFix  | 3/4 | 7/8 | 10/12  
FixMiner  | 2/3 | 12/14| 14/17  
LSRepair  | 8/14| 7/14 | 15/28  
SimFix  | 9/13| **14/26**| **23**/39  
kPAR  | 1/8 | 7/18 | 8/26  
AVATAR  | 5/11| 6/13 | 11/24  

MIMIC_opt | **11/19** | 10/25 | 21/**44**
MIMIC_all | 6/11 | 7/16 | 13/27  
MIMIC_top5 | 3/7 | 5/6 | 8/13  

## V. Generated Patches

*MIMIC* generates genuine/ plausible patches for 21/44 Defects4J bugs with its IR-based bug localizer. 


Patches | Genuineness
--------|---------
[Lang_6  ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Lang_6/)  |Correct 
[Lang_7  ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Lang_7/)  |Correct 
[Lang_10 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Lang_10/) |Correct 
[Lang_13 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Lang_13/) |Plausible 
[Lang_18 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Lang_18/) |Plausible 
[Lang_21 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Lang_21/) |Plausible 
[Lang_22 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Lang_22/) |Correct 
[Lang_24 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Lang_24/) |Correct 
[Lang_26 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Lang_26/) |Correct 
[Lang_33 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Lang_33/) |Correct 
[Lang_39 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Lang_39/) |Correct 
[Lang_43 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Lang_43/) |Plausible 
[Lang_44 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Lang_44/) |Plausible 
[Lang_45 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Lang_45/) |Plausible 
[Lang_47 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Lang_47/) |Correct 
[Lang_57 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Lang_57/) |Correct 
[Lang_58 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Lang_58/) |Plausible 
[Lang_59 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Lang_59/) |Correct 
[Lang_63 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Lang_63/) |Plausible 
[Math_2  ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Math_2/)  |Plausible 
[Math_5  ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Math_5/)  |Correct 
[Math_8  ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Math_8/)  |Plausible 
[Math_11 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Math_11/) |Correct 
[Math_15 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Math_15/) |Correct 
[Math_20 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Math_20/) |Plausible 
[Math_28 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Math_28/) |Plausible 
[Math_32 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Math_32/) |Plausible 
[Math_34 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Math_34/) |Correct 
[Math_35 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Math_35/) |Correct 
[Math_42 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Math_42/) |Plausible 
[Math_52 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Math_52/) |Plausible 
[Math_57 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Math_57/) |Correct 
[Math_58 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Math_58/) |Plausible 
[Math_59 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Math_59/) |Correct 
[Math_63 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Math_63/) |Plausible 
[Math_64 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Math_64/) |Plausible 
[Math_65 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Math_65/) |Correct 
[Math_70 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Math_70/) |Correct 
[Math_75 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Math_75/) |Correct 
[Math_82 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Math_82/) |Plausible 
[Math_95 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Math_95/) |Plausible 
[Math_96 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Math_96/) |Plausible 
[Math_97 ](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Math_97/) |Plausible 
[Math_104](https://github.com/fse19/mimic/blob/master/OUTPUT/MIMIC/FixedBugs/Math_104/)|Plausible 


## VI. Structure of the project
```powershell
  |--- README.md   :  user guidance
  |--- bin         :  binary code
  |--- d4j-info    :  defects4j information
  |--- doc         :  document
  |--- final       :  evaluation result
  |--- lib         :  dependent libraries
  |--- sbfl        :  fault localization tool
  |--- src         :  source code
  |--- test        :  test suite
```

----