# Job Monitor
###Version 1.1

Monitor the progress of your scripts in an HTML webpage.

Written by Andrew Schoen (schoen.andrewj@gmail.com)


###General Info:
This tool generates a webpage, some style sheets, some json files, and some scripts
which allow you to tag processes across or within other scripts, and monitor their progress.

Most of the setup is done for you, but there are still a few things you will
need to do to fully implement the system. 

###Setup:

First, to arrange things, you will need to create two simple CSV files, one for "Jobs", and
the other for "Events". Simply put, "Jobs" will become the rows in your resulting webpage table,
while "Events" will become the columns. There are two headers in each CSV file: "ID" and "NAME".
"ID" is the name that your code uses, and the web page uses as an indicator for that category, 
whereas "NAME" is a front-end version of the ID, with prettier formating. Here is an example

Jobs CSV:

| ID   | NAME       |
|------|------------|
| SUB1 | Subject 1  |
| SUB2 | Subject 2  |
| SUB3 | Subject 3  |

Events CSV:

| ID    | NAME    |
|-------|---------|
| Step1 | Step 1  |
| Step2 | Step 2  |
| Step3 | Step 3  |

The resulting HTML table would look something like this:

| JOB  | Step 1 | Step 2 | Step 3 | Step 4 | Step 5 |
|------|--------|--------|--------|--------|--------|
| SUB1 | Status | Status | Status | Status | Status |
| SUB2 | Status | Status | Status | Status | Status |
| SUB3 | Status | Status | Status | Status | Status |

Each Status can be "Inactive", "Running", "Finished", or "Error".

###Usage
```
SetupJobMonitor.py [options] <process_name> <monitor_dir> <jobs_file> <events_file>
```
Arguments:

| Arguments      | Explanation |
|----------------|-------------|
| process_name | The name of the online monitor page. The resulting HTML file will be a cleaned (url-friendly) version of this name. |
| jobs_file    | The output directory for the monitor html and components. Ideally, this would be in a public location, such as a pub_html directory. |
| events_file  | A csv file (.csv) containing two columns, with headers ID and NAME. I column of unique job identifiers. In most cases, there would be a single job for each scan/subject. ID is a short unique reference to the job, with no spaces. Name is a pretty version of the ID. |
| monitor_dir  | A csv file (.csv) containing two columns, with headers ID and NAME. I column of unique event identifiers. ID is a short unique reference to the event with no spaces. Name is a pretty version of the ID. These can be as fine-grained as you would like (within or across scripts), but denote the separate steps each job goes through. |

Options:

| Options      | Explanation          |
|--------------|----------------------|
| -h --help    | Show the help screen |
| -v --version | Show the version     |

###Implementing

Once you run the setup python script, you will need to implement the code to call it in your
own programs. That is explained below:

Say you have a bash script like so:
```
#!/bin/bash                              
#Some simple script by me called step1.sh

subj=$1                                  
someprogram -option $subj arg1 arg2      
```

We want to make the Job Monitor know how the script above runs. We will first want to indicate to
the Job Monitor that our script is running. Imagine that our step1.sh corresponds with the first
event column in our job monitor, and subj is our job identifier. The code we will run for the example
above would look something like:

```
path/to/statusupdate.py $subj step1 Running
```

Running that command will update the json file associated with the subject/job, and the webpage will
poll that json periodically to update. Refreshing manually will ensure the most up-to-date version is
presented.

Now assume we want to update the Job Monitor when our job finishes. The simple way would be to run the
following command after executing "someprogram":

```
path/to/statusupdate.py $subj step1 Finished
```

Combining these two snippets with our original bash script gives us:

```
#!/bin/bash                                 
#Some simple script by me called step1.sh   

subj=$1                                     
path/to/statusupdate.py $subj step1 Running 
someprogram -option $subj arg1 arg2         
path/to/statusupdate.py $subj step1 Finished
```

However, we may want to know if "someprogram" ran successfully, or if it had an error.
Many programs will issue a return value, which can be captured in an "if statement".
Consult the documentation on the command to see if this is the case. If so, you can wrap
your someprogram in the if statement as so:

```
#!/bin/bash                                   
#Some simple script by me called step1.sh     

subj=$1                                       
path/to/statusupdate.py $subj step1 Running   

if someprogram -option $subj arg1 arg2 ; then 
  path/to/statusupdate.py $subj step1 Finished
else                                          
  path/to/statusupdate.py $subj step1 Error   

```

Alternatively, you may be able to check for the existence of some output, as so:

```
#!/bin/bash
#Some simple script by me called step1.sh

subj=$1                                       
path/to/statusupdate.py $subj step1 Running   

someprogram -option $subj arg1 arg2

if [[ -e someoutput ]] ; then
  path/to/statusupdate.py $subj step1 Finished
else
  path/to/statusupdate.py $subj step1 Error
```

If you have questions, please contact Andrew Schoen at schoen.andrewj@gmail.com . Enjoy, and Happy Monitoring!
