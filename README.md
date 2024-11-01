

lab_eval  evaluate python scripts submitted by student. 
The use case is as follows: 
Students have top complete a laboratory that includes python scripts. 
Each students submits the scripts and a report using (for example) moodle. 

This scripts takes the submission of the students and test automatically the scripts against those of the instructor. 
The evaluation results in a grade being assigned to each student as well as a log file that can be helpful to the student for debugging. 

# Installation

```
$ git clone 
$ cd lab_eval
$ python3 -m build && pip3 install --force-reinstall dist/pylab_eval-0.0.<version>.tar.gz
```

# Usage Example

## Evaluating a single Labs

Evaluation of the caesar of the student

```
lab_eval_lab  --conf ~/gitlab/lab_eval/examples/lab_caesar.cfg ./labs/Firstname,_Lastname_5698747_assignsubmission_file_
```
In order to specify the output log file (toto.log):
Note that we assume that the lab_caesar.cfg specifies that output is placed in a log file by specifying the `log_dir`. This variable is usually commented, in which case, either we should uncomment it or specify it in the command line using `--log_dir <log_dir>`.

```
lab_eval_lab  --conf ~/gitlab/lab_eval/examples/lab_caesar.cfg --lab_id toto ./labs/Firstname,_Lastname_5698747_assignsubmission_file_
```
## Evaluation the class 

The configuration file is located in ~/gitlab/lab_eval/examples/lab_caesar.cfg and specifies the various locations of the necessary files:

* the instructir's python files (the solutions)
* the file that evaluates the student's scripts against the solutions lab_caesar.py
* the student database. The student data base is provided by moodle in "Participants" tab, "select all participants" export as JSON file.
* the labs submitted by the students "Consulted assignments" > "Download all assignments" - in our case  INF808-AI-Laboratoire1\ Attaques\ cryptographiques\ \(Remise\ des\ devoirs\)-2622297.zip


To run the evaluation from the moodle file


```
lab_eval_class  --conf ~/gitlab/lab_eval/examples/lab_caesar.cfg  --class_dir lab1   INF808-AI-Laboratoire1\ Attaques\ cryptographiques\ \(Remise\ des\ devoirs\)-2622297.zip 
```

In general, we expect that some question are evaluated manually. This can occurs when the lab combines code evaluation as well as a report evaluation. Once the report has been evaluated and the associated scores placed into the `score_list.json` file, we recompute the grades to consider the manually added scores as follows:

```
lab_finalize_grades  --conf ~/gitlab/lab_eval/examples/lab_caesar.cfg test_class_dir/score_list.json

```
## Adding to score_files

The evaluation of the class evaluates the python scripts and generates a file `score_list.json` file. In our case, in addition to the python scripts we are willing to evaluate the report of the lab - which is a PDF file. 
One way to do, is to edit the `score_list.json` file directly and add the grade associated to the report. 
Another way to proceed is to evaluate the PDF report in a separate score_list file `pdf_score_list.json`, and then combine the two files together. To do so we proceed as follows:

```
lab_add_score_list --conf ~/gitlab/lab_eval/examples/lab_caesar.cfg score_list.json pdf_score_list.json 
```

## Generating the grades


```
lab_finalize_grades --conf ~/gitlab/lab_eval/examples/lab_caesar.cfg score_list.json 
```

## Exporting in xls

To fill the scores to the University, one needs to complete a xls sheet. 
The following function extracts the list of students id in a column. Then the corresponding grades are generated before being included again back in the original xls sheet. 


```
lab_export_xls --conf ~/gitlab/lab_eval/examples/lab_caesar.cfg   --sheet_name lab1 score_list.json  notes-INF808Gr18-A2023.xlsx 

```
There is still a need to manually copy paste the clumns of the grade_lab1.xls file. The reason is that we have not been able to complete the xls file. Further improvement may consider adding a sheet to the original file or using the concatenating of the dataframes. 


# Real Example

## lab_caesar

We download the scripts from moodle and run the evaluation. 

```
lab_eval_class  --conf ~/gitlab/lab_eval/examples/lab_caesar.cfg  --class_dir lab1   INF808-AI-Laboratoire1\ Attaques\ cryptographiques\ \(Remise\ des\ devoirs\)-2622297.zip
```

This creates a directory lab1 in which we can find the resulting evaluation `score_list.json`. However, we received some scripts via email. We currently cannot re-run the evaluation and the existing expanded zip file. The main reason is that if we enable this, it remains confusing what is re-evaluated, what is overwritten. As aresult, we need to build a new zip file. 
To do so, we place the up-todate scripts in the `lab1/lab` directory under the right student, selecte all student directories and compress them in zip file `moodle.zip` and re-run the evaluation.
```
lab_eval_class  --conf ~/gitlab/lab_eval/examples/lab_caesar.cfg  --class_dir lab1 moodle.zip 
```

The lab_caesar combines the evaluation of the scripts and a report. As a result, we have the evaluation of the report performed in `pdf_score_list.json` and the evaluation of the scripts in `lab1/score_list.json`. To combine these two score_list we add those of the report to those of the scripts with the following command line.
Note the added file is the second argument so `pdf_score_list.json`` will be added to `lab/score_list.json`.

```
lab_add_score_list --conf ~/gitlab/lab_eval/examples/lab_caesar.cfg lab1/score_list.json pdf_score_list.json
```

Note: We probably need to define a function that sets to None a list. This enables to initialize the pdf_score_list.json from score_list.json. 
We can also check that 'grade (total)' and 'grade(%)' have been updated.  

```
import json 
with open( 'pdf_score_list.json', 'rt', encoding='utf8' ) as f:
  l = json.loads( f.read( ) )
for k in l.keys() :
  for q in l[ k ].keys():
    try:
      int( q )
      l[ k ][ q ] = None
    except:
      continue
with open( 'pdf_score_list.json', 'wt', encoding='utf8' ) as f:
  f.write( json.dumps( l ),  indent=2 ) 
```


When original files have been saved, we can check `pdf_score_list.json` has not been modified while `lab1/score_list.json` has been modified. 

We need then to generate the xls files that are considered as input to their system Genote. They provide an xls file to be complete. The column that contains the student id is located at row 17 column 0. The following function takes that column and generates the corresponding grades in a file `lab1_grades.xls`. We need then to manually copy/paste the grades to the main file before submitting. 
In our case, we have two groups so we run the following commands:

```
lab_export_xls --conf ~/gitlab/lab_eval/examples/lab_caesar.cfg --student_id_row 17 --student_id_col 0  --sheet_name lab1_18 lab1/score_list.json  notes-INF808Gr18-A2023.xlsx
lab_export_xls --conf ~/gitlab/lab_eval/examples/lab_caesar.cfg --student_id_row 17 --student_id_col 0  --sheet_name lab1_19 lab1/score_list.json  notes-INF808Gr19-A2023.xlsx
```

## Intra or Final (Entirely performed on Moodle and exported in json)

In this case, the intra is exclusively performed in moodle and we export the results in a json format. That file does not match exactly the format we are using for 'score_list.json'. We could have used the xls format. However, we encoutered a few difficulties described as follows:

* Numbers are using a "2,3" format instead of "2.3" which makes math operations such as addition difficult. In our case, we want to update some questions for exemple that had an error. 
* We have all students in one file, while we need to import the results per sub groups. 

To convert the json file exported from moodle, we use the following command:
`max_total_score` is necessary

There are several possible json_files. In our case, the one we consider is the one with all question results. This means "point of each question" needs to be checked.

```
lab_moodle_to_score_list --max_total_score 37 INF808-AI-Intra-notes.json
```

The resulting file is 'score_list.json'

We eventually perform some operations. 

```
lab_export_xls --student_id_row 14 --student_id_col 0  --sheet_name intra_inf808_18 score_list.json  notes-INF808Gr18-A2023.xlsx
lab_export_xls --student_id_row 14 --student_id_col 0  --sheet_name intra_inf808_19 score_list.json  notes-INF808Gr19-A2023.xlsx
lab_export_xls --student_id_row 14 --student_id_col 0  --sheet_name intra_ift511_1 score_list.json  notes-IFT511Gr1-A2023.xlsx
```

## Lab2 

We update directly the json file. In this case, we provides the points to anyone that has given a 'decent' report as most of the information was needed to execute the code. 
 
```
import json
with open( './score_list.json_script_report', 'rt', encoding='utf8' ) as f:
  scores = json.loads( f.read() )
  
  
for k in scores.keys():
  if k != "student_id_with_no_report":
    scores[ k ][ '2' ] = 1 
    scores[ k ][ '5' ] += 1
    scores[ k ][ '9' ] = 1 
  else: 
    scores[ k ][ '2' ] = 0
    scores[ k ][ '9' ] = 0

with open( './score_list.json_script_report', 'wt', encoding='utf8' ) as f:
  f.write( json.dumps( scores, indent=2 ) )
```

To update the grades:
```
lab_finalize_grades --conf ~/gitlab/lab_eval/examples/lab_babyesp.cfg score_list.json_script_report 
```

To export to genote:

```
lab_export_xls --student_id_row 14 --student_id_col 0  --sheet_name lab2_inf808_18 score_list.json_script_report  notes-INF808Gr18-A2023.xlsx
lab_export_xls --student_id_row 14 --student_id_col 0  --sheet_name lab2_inf808_19 score_list.json_script_report  notes-INF808Gr19-A2023.xlsx
lab_export_xls --student_id_row 14 --student_id_col 0  --sheet_name lab2_ift511_1 score_list.json_script_report  notes-IFT511Gr1-A2023.xlsx
```

## Individual submissions

When one studnet missed the dead line, we may want to add its contributions individualy. One way to do so is to rebuild the moodle.zip file. 

If the evaluation has already run over moodle.zip, all files are unzipped under the lab1/cryptolab/moodle_dir directory. This means that we have:

```
lab1/cryptolab/moodle_dir/moodle_dir
  +- student_dir_start
  ...
  +- late_student_dir
  ...
  +- student_dir_end
``` 

When the files of the late student are updated, we run

```
cd lab1/cryptolab/moodle_dir/moodle_dir
zip -r "/home/mglt/uds/2024-fall-inf808/lundi/lab1/INF808-AL-Laboratoire Attaques cryptographiques Introduction aux attaques cryptographiques-3001267.zip"  .
```

# Configuration 

## Configuration file

Check the `lab_eval.git/examples/lab_caesar.cfg` as an example.

## StudentDB file



# Building the module to evaluate your own lab

please check `lab_eval.git/examples/lab_caesar.cfg` or `lab_eval.git/examples/lab_babyesp.cfg`


# Other links:

Here are some additional resources to push further the automation of Moodle labs assignaments:

https://github.com/troeger/moodleteacher
https://github.com/hexatester/moodlepy/blob/master/moodle/mod/assign/assignment.py

# TODO

* refine the necessary operations: between moodle / genote and labeval.
* max_score MUST always be provided when using finalize. In many cases, this value is taken from the Evalpy module. We may consider adding it as a metadata in score_list or in a conf file or as an argument to each command line. lab_finalize_grades lab_add_score_list. 
* column description for genote excel files may be part of a conf file. 
* initializations of scoreFile list needs to be revised   init_from_file needs a self. 

