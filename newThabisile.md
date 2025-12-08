       * simple cobol example for DLP source code detection
       IDENTIFICATION DIVISION.
       PROGRAM-ID. AUDITPROG.
       ENVIRONMENT DIVISION.
       DATA DIVISION.
       WORKING-STORAGE SECTION.
       01  CUSTOMER-NUMBER        PIC 9(10) VALUE 0000000001.
       01  CUSTOMER-NAME          PIC X(20)  VALUE "ABSA-AUDIT".
       PROCEDURE DIVISION.
           DISPLAY "HELLO AUDIT".
           MOVE 12345 TO CUSTOMER-NUMBER.
           PERFORM VARYING CUSTOMER-NUMBER FROM 1 BY 1 UNTIL CUSTOMER-NUMBER > 10
               DISPLAY "COUNT " CUSTOMER-NUMBER
           END-PERFORM
           STOP RUN.

       *> -----------------------------------------------------------------
       *>  AUDITDLP2.cbl  —  DLP source code test (table + control flow)
       *> -----------------------------------------------------------------
        IDENTIFICATION DIVISION.
        PROGRAM-ID. AUDITDLP2.

        DATA DIVISION.
        WORKING-STORAGE SECTION.
        01  WS-IX                         PIC 9(3) VALUE 1.
        01  WS-MAX                        PIC 9(3) VALUE 5.
        01  STATUS-TABLE.
            05  STATUS-ENTRY OCCURS 5 TIMES.
                10  CODE                  PIC X(1).
                10  MEANING               PIC X(20).
        01  SEARCH-CODE                   PIC X(1) VALUE 'A'.
        01  FOUND-IX                      PIC 9(3) VALUE 0.

        PROCEDURE DIVISION.
        INIT.
            MOVE 'A' TO CODE (1)
            MOVE 'ACTIVE'         TO MEANING (1)
            MOVE 'I' TO CODE (2)
            MOVE 'INACTIVE'       TO MEANING (2)
            MOVE 'S' TO CODE (3)
            MOVE 'SUSPENDED'      TO MEANING (3)
            MOVE 'P' TO CODE (4)
            MOVE 'PENDING'        TO MEANING (4)
            MOVE 'X' TO CODE (5)
            MOVE 'UNKNOWN'        TO MEANING (5)

        LOOP-AND-SEARCH.
            PERFORM VARYING WS-IX FROM 1 BY 1 UNTIL WS-IX > WS-MAX
                EVALUATE CODE (WS-IX)
                    WHEN 'A'
                        CONTINUE
                    WHEN 'I'
                        CONTINUE
                    WHEN OTHER
                        CONTINUE
                END-EVALUATE
            END-PERFORM

            SEARCH STATUS-ENTRY
                AT END MOVE 0 TO FOUND-IX
                WHEN CODE (INDEX STATUS-ENTRY) = SEARCH-CODE
                    MOVE INDEX STATUS-ENTRY TO FOUND-IX
            END-SEARCH

            STOP RUN.

       *> CUSTOMER-REC.cpy — record layout copybook (synthetic)
        01  CUSTOMER-RECORD.
            05 CUSTOMER-ID                PIC 9(10).
            05 CUSTOMER-NAME              PIC X(30).
            05 CUSTOMER-STATUS            PIC X(1).

