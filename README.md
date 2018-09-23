# utl-import-excel-workbooks-in-all-folders-and-subfolders
Import excel workbooks in all folders and subfolders.  Keywords: sas sql join merge big data analytics macros oracle teradata mysql sas communities stackoverflow statistics artificial inteligence AI Python R Java Javascript WPS Matlab SPSS Scala Perl C C# Excel MS Access JSON graphics maps NLP natural language processing machine learning igraph DOSUBL DOW loop stackoverflow SAS community.

    Import excel workbooks in all folders and subfolders

    see github
    https://tinyurl.com/y8f87bo2
    https://github.com/rogerjdeangelis/utl-import-excel-workbooks-in-all-folders-and-subfolders

    see SAS Forum
    https://tinyurl.com/yardkcz7
    https://communities.sas.com/t5/SAS-Programming/Integrate-Folder-scanning-code-with-proc-import-for-multiple/m-p/498183

    11 directory utilties
    https://tinyurl.com/yaajbg36
    https://github.com/rogerjdeangelis/utl_file_and_directory_utilities_for_all_operating_systems


    INPUT
    =====

       C:\parent
       |
       +  par.xlsx
       |
       +---child_1
       |  |
       |  +parch1.xlsx
       |
       \---child_2
          |
          +parch2.xlsx


    EXAMPLE OUTPUT (log and three datasets)
    ---------------------------------------

     WORK.LOG total obs=3

       PTH                              SHEET                STATUS

       c:/parent/par.xlsx               par       workbook imported sucessfully
       c:/parent/child_1/parch1.xlsx    parch1    workbook imported sucessfully
       c:/parent/child_2/parch2.xlsx    parch2    workbook imported sucessfully


     WORK.PAR total obs=19

      NAME       SEX    AGE    HEIGHT    WEIGHT

      Alfred      M      14     69.0      112.5
      Alice       F      13     56.5       84.0
      Barbara     F      13     65.3       98.0
      Carol       F      14     62.8      102.5
      Henry       M      14     63.5      102.5
     ...
     ...

     WORK.PARCH2 total obs=19

      NAME       SEX    AGE    HEIGHT    WEIGHT

      James       M      12     57.3       83.0
      Jane        F      12     59.8       84.5
      Janet       F      15     62.5      112.5
      Jeffrey     M      13     62.5       84.0
      John        M      12     59.0       99.5
      Joyce       F      11     51.3       50.5


    PROCESS
    =======

       proc datasets lib=work kill;
       run;quit;

      * directory and subdirectory file;
      data dir;
         length root path $200 dir 8;
         call missing(path,dir);
         input root;
       cards;
       c:/parent
       ;run;

       data dir;
         modify dir;
         rc=filename('tmp',catx('/',root,path));
         dir=dopen('tmp');
         replace;
         if dir;
         path0=path;
         do _N_=1 to dnum(dir);
           path=catx('/',path0,dread(dir,_N_));
           output;
           end;
         rc=dclose(dir);
       run;quit;

       /*
       WORK.DIR total obs=9

          ROOT       PATH                 DIR

          ROOT       PATH                   DIR

        c:/parent                            1
        c:/parent    child_1                 1
        c:/parent    child_2                 1

        c:/parent    par.xlsx                0
        c:/parent    child_1/parch1.xlsx     0
        c:/parent    child_2/parch2.xlsx     0
       */


       %symdel pth sheet cc / nowarn;

       * Import workbooks;
       data log;

          length pth $64;

          set dir(where=(index(path,'xlsx')>0));

          pth=cats(root,'/',path);
          sheet=scan(path,-2,'./');

          call symputx('pth',pth );
          call symputx('sheet',sheet);

          rc=dosubl('
             libname pth "&pth";
             data &sheet;
                set pth.&sheet;
             run;quit;
             %let cc=&syserr;
             libname pth clear;
          ');

          if symgetn('cc')=0 then status="workbook imported sucessfully";
          else status = "workbook import failed";

          drop dir rc root path;

       run;quit;

       proc contents data=work._all_;
       run;quit;


    OUTPUT see above
    ================


              Member   Obs, Entries
     Name     Type      or Indexes   Vars

     DIR      DATA           6        3
     LOG      DATA           3        3

     PAR      DATA          19        5  from workbooks
     PARCH1   DATA          19        5
     PARCH2   DATA          19        5

    *                _               _       _
     _ __ ___   __ _| | _____     __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \   / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/  | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|   \__,_|\__,_|\__\__,_|

    ;


     * create folders subfolders;
     data _null_;

         * create directory;
         if _n_=0 then do;
             %let rc=%sysfunc(dosubl('
                data _null_;
                    rc=dcreate("parent","c:/");
                    rc=dcreate("child_1","c:/parent");
                    rc=dcreate("child_2","c:/parent");
                run;quit;
            '));
         end;

    run;quit;

    * put excel files in folders and subfolders;

    libname par "c:/parent/par.xlsx";
    libname parch1 "c:/parent/child_1/parch1.xlsx";
    libname parch2 "c:/parent/child_2/parch2.xlsx";

    data par.par parch1.parch1 parch2.parch2;
       set sashelp.class;
    run;quit;


