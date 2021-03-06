# utl_five_algorithms_to_split_a_table_based_on_a_categorical_variable
Five algorithms to split a table based on a categorical variable. 
    Five algorithms to split a table based on a categorical variable

    github
    https://goo.gl/wD3EkA
    https://github.com/rogerjdeangelis/utl_five_algorithms_to_split_a_table_based_on_a_categorical_variable

      Five Solutions

         1. DOSUBL SQL Datastep
         2. DOSUBL Sort Datastep
         3. DOSUBL Just datastep
         4. R SPLIT function
         5. HASH SORT thne HASH
    see
    https://goo.gl/4QL1o7
    https://communities.sas.com/t5/SAS-Statistical-Procedures/How-to-quot-Split-Data-quot-By-Group-Processing/m-p/431439

    see
    Novinosrin profile
    https://communities.sas.com/t5/user/viewprofilepage/user-id/138205


    INPUT
    =====

     WORK.HAVE total obs=27                 |  RULES  (NW AND ow DATASETS)
                                            |
       WEIGHT    ID    TREATMENT    KCAL    |  WORK.NW total obs=15
                                            |
         NW      1         A         400    |    WEIGHT    ID    TREATMENT    KCAL
         NW      2         A         500    |
         OW      3         A         560    |      NW      1         A         400
         NW      4         A         800    |      NW      2         A         500
         OW      5         A         490    |      NW      4         A         800
         NW      6         A         500    |      NW      6         A         500
         OW      7         A         400    |    ...
                                            |  WORK.OW total obs=12
                                            |
                                            |    WEIGHT    ID    TREATMENT    KCAL
                                            |
                                            |      OW      3         A         560
                                            |      OW      5         A         490
                                            |      OW      7         A         400


    PROCESS ( All the code)
    =======================

     1. DOSUBL
     =========

        data _null_;

          * faster than distinct;
          if _n_=0 then do;
            %let rc=%sysfunc(dosubl('
              proc sql;
                 select quote(max(weight)) into :wgts separated by "," from have  group by weight
              ;quit;
            '));
          end;

          do wgt=&wgts;
            call symputx("wgt",wgt);
            rc=dosubl('
              data &wgt;
                set have(where=(weight=symget("wgt")));
              run;quit;
            ');
          end;

        run;quit;

     2. SORT DOSUBL (need an idex on weight)
     =======================================

        data _null_;

          if _n_=0 then do;
            %let rc=%sysfunc(dosubl('
              proc sort data=have out=havSrt;
                 by weight;
              run;quit;
            '));
          end;

          set havSrt(keep=weight);
          by weight;
          if last.weight then do;
             call symputx('wgt',weight);
             rc=dosubl('
                data &wgt;
                   set have(where=(weight=symget("wgt")));
                run;quit;
            ');
          end;
        run;quit;


     2. DOSUBL JUST DATASTEP
     =======================

        * just in case they exist;
        proc datasets lib=work;
          delete ow nw rec;
        run;quit;

        data _null_;

          set have(keep=weight);
          call symputx('wgt',weight);
          call symputx('rec',_n_);

          * need to optimizing compiler;
          rc=dosubl('
            data rec; set have; if _n_=&rec then do; output;stop;end; run;quit;
            proc append base=&wgt data=rec;
            run;quit;
          ');

        run;quit;


     4. R SPLIT FUNCTION
     ===================

        %utl_submit_wps64('
        libname sd1 sas7bdat "d:/sd1";
        options set=R_HOME "C:/Program Files/R/R-3.3.2";
        libname wrk sas7bdat "%sysfunc(pathname(work))";
        proc r;
        submit;
        source("C:/Program Files/R/R-3.3.2/etc/Rprofile.site", echo=T);
        library(haven);
        have<-read_sas("d:/sd1/have.sas7bdat");
        want<-split(have, list(have$WEIGHT), drop = TRUE);
        list2env(want,envir=.GlobalEnv);
        endsubmit;
        import r=OW  data=wrk.wantwpsow;
        import r=NW  data=wrk.wantwpsnw;
        run;quit;
        ');

        proc print data=wantwpsnw(obs=4);
        run;quit;

        Up to 40 obs from wantwpsnw total obs=15

        Obs    WEIGHT    ID    TREATMENT    KCAL

          1      NW      1         A         400
          2      NW      2         A         500
          3      NW      4         A         800
          4      NW      6         A         500

      5. HASH
      ========

        proc sort date=have;
          by weight;
        run;quit;

        data _null_;
        if _n_=1 then do;
           if 0 then set have;
           dcl hash h(dataset:'have(obs=0)',multidata:'y');
           h.definekey('Weight');
           h.definedata(all:'y');
           h.definedone();
         end;
         set have;
            by weight;
            if first.weight then h.clear();
            h.add();
            if last.weight then h.output(dataset: weight);
         run;quit;

    *                _              _       _
     _ __ ___   __ _| | _____    __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \  / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/ | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|  \__,_|\__,_|\__\__,_|

    ;


    options validvarname=upcase;
    libname sd1 "d:/sd1";
    data sd1.have have;
    informat weight $2.;
    input ID $ Weight $ Treatment $ kcal;
    datalines;
    1 NW A 400
    2 NW A 500
    3 OW A 560
    4 NW A 800
    5 OW A 490
    6 NW A 500
    7 OW A 400
    8 OW A 700
    9 NW A 900
    1 NW B 580
    2 NW B 600
    3 OW B 800
    4 NW B 500
    5 OW B 600
    6 NW B 800
    7 OW B 700
    8 OW B 500
    9 NW B 780
    1 NW C 570
    2 NW C 670
    3 OW C 570
    4 NW C 400
    5 OW C 600
    6 NW C 800
    7 OW C 800
    8 OW C 500
    9 NW C 800
    ;;;;
    run;quit;

    *                  _
     _ __   __ _ _   _| |
    | '_ \ / _` | | | | |
    | |_) | (_| | |_| | |
    | .__/ \__,_|\__,_|_|
    |_|
    ;

    Paul Dorfman <sashole@bellsouth.net>
    1:58 AM (6 hours ago)

     to SAS-L, me
    Roger,

    With hashing against the sorted data, it's most natural to use the DoW-loop:

    data _null_;
      dcl hash h(dataset:'have(obs=0)',multidata:'y');
       h.definekey('Weight');
       h.definedata(all:'y');
       h.definedone();
       do until (last.weight) ;
          set have;
          by weight;
          h.add();
       end ;
        h.output(dataset: weight);
     run;

    However, when you use the hash method, you don't have to sort beforehand.
    Instead, you can have as many instances of object H as there are distinct
    values of Weight, creating them on the fly as you encounter new values of
    Weight and storing the pointers to them by Weight in another hash table
    (X, below). While you're reading the file, you store the data for each value of Weight in
    the corresponding instance of H, and when the file has been processed,
    iterate through each instance and dump its data content to the aptly named file, like so:

    data _null_ ;
      dcl hash x() ;
      x.defineKey  ('weight') ;
      x.defineData ('weight','h') ;
      x.defineDone () ;
      dcl hash h ;
      do until (z) ;
        set have end = z ;
        if x.find() ne 0 then do ;
          h = _new_ hash (dataset:'have(obs=0)', multidata:'y') ;
          h.defineKey  ('weight') ;
          h.defineData (all:'y') ;
          h.defineDone () ;
          x.add() ;
        end ;
        h.add() ;
      end ;
      dcl hiter i('x') ;
      do while (i.next()=0) ;
        h.output (dataset: weight) ;
      end ;
      stop ;
    run ;

    Credit for inventing the method goes, of course, to Richard DeVenezia (2004).
     After my hash talk at Montreal SUGI, I was asked if hash object instances
    could be stored as data in another hash table. I replied "Not to my knowledge"
    and was wrong, for soon after the conference, Rich posted code akin to the
    above on SAS-L. Well, I wasn't entirely wrong because it's not really the inst
    ances of H stored in X as data but the pointers to them. Every time X is
    successfully searched using X.FIND(), a new value of non-scalar variable H
    (of type hash object) is surfaced in the PDV by overwriting the host variable
    H. It points to the instance to which the data is then added by H.ADD().
    The collection process works similarly. The iterator I dumps the current pointer valu
    e of H into the PDV, and this is what gets picked up by the H.OUTPUT method.



