# utl-sometimes-you-need-a-fat-analysis-table-although-rarely
    Sometimes you need a fat analysis table although rarely

    Nicely laid out example of why a fat table is needed?

    I take back the fat table statement

    See additional solution by PGStats on end
    https://communities.sas.com/t5/user/viewprofilepage/user-id/462

     As a side note
       Paul got me thinking. Why can't some SAS procs output a view?
       Only a massive sort breaks the pipe. Presorted with by does not?

    see github
    https://tinyurl.com/ybmqrrkm
    https://github.com/rogerjdeangelis/utl-sometimes-you-need-a-fat-analysis-table-although-rarely

    SAS Forum
    https://tinyurl.com/yb57e697
    https://communities.sas.com/t5/SAS-Programming/Computation-using-loop/m-p/511846

    INPUT
    =====

    WORK.HAVE total obs=8

       RANGE      COL      VAL  |     RULES
                                |
     1992-1995    B0       0.2  |     B0=0.2
     1992-1995    B1      25.0  |     B1=0.5*B0 + B1  = 25.1
     1992-1995    B2      30.0  |     B2=0.5*B1 + B2  = 12.55 + 30 = 42.55
     1992-1995    B3      40.0  |     B3=0.5*B2 + B3  = 21.275 + 40= 61.275

     2002-2005    B2       0.3  |     B0=0.3
     2002-2005    B3      50.0  |     B1=0.5*B0 + B1
     2002-2005    B0     100.0  |     B2=0.5*B1 + B2
     2002-2005    B1     150.0  |     B3=0.5*B2 + B3

    EXAMPLE OUTPUT
    --------------

     WORK.WANT total obs=2

       RANGE       B0      B1        B2         B3

     1992-1995    0.2    25.10     42.550     61.275
     2002-2005    0.3    50.15    125.075    212.538


    PROCESS
    =======

    data want;

      if _n_=0 then do; %let rc=dosubl('
         proc transpose data=have out=havXpo(drop=_name_);
          by range;
           var val;
           id col;
         run;quit;

         '));
      end;

      set havXpo;

      B1=0.5*B0 + B1;
      B2=0.5*B1 + B2;
      B3=0.5*B2 + B3;

    run;quit;

    *                _              _       _
     _ __ ___   __ _| | _____    __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \  / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/ | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|  \__,_|\__,_|\__\__,_|

    ;

    * I made slight chages to your input;
    data have (keep=range col val);
     retain cnt -1 range col;
     input yer var$ val;
     cnt=cnt+1;
     col=cats('B',mod(cnt,4));
     if var ne lag(var) then range=cats(yer,'-',yer+3);
    cards4;
    1992 A 0.2 .
    1993 A 25
    1994 A 30
    1995 A 40
    2002 B 0.3 .
    2003 B 50
    2004 B 100
    2005 B 150
    ;;;;
    run;quit;



    *____   ____ ____  _        _
    |  _ \ / ___/ ___|| |_ __ _| |_ ___
    | |_) | |  _\___ \| __/ _` | __/ __|
    |  __/| |_| |___) | || (_| | |_\__ \
    |_|    \____|____/ \__\__,_|\__|___/

    ;

    I take back my solution see

    PGstats
    https://communities.sas.com/t5/user/viewprofilepage/user-id/462

    data have;
    input year firm $ factor A;
    datalines;
    1992  A 0.2 .
    1993  A 0.5 25
    1994  A 0.5 30
    1995  A 0.5 40
    2002  B 0.3 .
    2003  B 0.5 50
    2004  B 0.5 100
    2005  B 0.5 150
    ;

    data want;
    set have;
    by firm;
    if first.firm then B = factor;
    else B = factor*prevB + A;
    retain prevB;
    prevB = B;
    drop prevB;
    run;

    proc print data=want noobs; run;

       year    firm    factor     A           B

       1992     A        0.2       .      0.200
       1993     A        0.5      25     25.100
       1994     A        0.5      30     42.550
       1995     A        0.5      40     61.275
       2002     B        0.3       .      0.300
       2003     B        0.5      50     50.150
       2004     B        0.5     100    125.075
       2005     B        0.5     150    212.538






