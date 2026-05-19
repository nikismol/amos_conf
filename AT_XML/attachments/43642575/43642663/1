select row_number() over(ORDER BY h.ac_registr, h.event_perfno_i) num
     , a.ac_typ
	 , h.event_perfno_i wo
     , h.ac_registr
     , case when state = 'O' then 'Open'
            when state = 'C' then 'Closed'
            else state 
        end status
     , decode(issue_date,
                0, 
                null, 
                to_char(to_date('31.12.1971', 'dd.mm.yyyy') + issue_date, 
                        'dd.mm.yyyy')) issue_date
     , absolute_due_date
     , absolute_due
     , decode(expected_date,
                    0, 
                    null, 
                    to_char(to_date('31.12.1971', 'dd.mm.yyyy') + expected_date, 
                            'dd.mm.yyyy')) due_date
     , ata_chapter
     , case when type = 'P' then 'Pirep'
            when type = 'C' then 'Cabin'
            when type = 'M' then 'Maintenance'
            when type = 'S' then 'Sched.'
            when type = 'T' then 'Templ.'
            when type = 'PD' then 'Pend.'
        else 'Unk. type'
    end type
	 , case when part_req_status = 'NN' then 'NR'
           when part_req_status in ('UN', 'UP', 'U?') then 'U'
           when part_req_status in ('AN', 'AQ', 'AP', 'UC', 'A?') then 'A'
           when part_req_status = 'DO' then 'AC'
           when part_req_status in ('TN', 'TP') then 'TR'
           when part_req_status = 'PN' then 'NA'
           when part_req_status = 'EN' then 'A+E'
        else 'Unk.'
    end parts
    , case when exists (select * from db_link where db_link.source_pk = to_char(h.event_perfno_i) ) then 'Y'
        else ' '
    end ref
    , case when mel_code <> 'В ' then 'M: ' || mel_code
        else ' '
    end mel
    , case when hil = 'Y' then 'DD'
        else ' '
    end dd
    , initcap(lastname) || ' ' || initcap(firstname) || ' (' || employee_no || ')' iss
    , h.projectno
    , wpno
    , case when start_date > 0 then 
            decode(start_date,
                    0, 
                    null, 
                    to_char(to_date('31.12.1971', 'dd.mm.yyyy') + start_date, 
                            'dd.mm.yyyy')) || ' (' || wh.station || ')' 
        else ' ' 
    end wp_planned_station
    , d.text wo_descr
    from wo_header h
    join forecast f on h.event_perfno_i = f.event_perfno_i
    join aircraft a on h.ac_registr = a.ac_registr
    join wo_transfer t on h.event_perfno_i = t.event_perfno_i
    join sign s on h.issue_sign = s.user_sign
    left join wp_assignment wa on h.event_perfno_i = wa.event_perfno_i
    left join wp_header wh on wa.wpno_i = wh.wpno_i
    join workstep_link s on h.event_perfno_i = s.event_perfno_i and sequenceno = 1
    join wo_text_description d on s.descno_i = d.descno_i
    left join ( select event_perfno_i, listagg (absolute_due, ' ') within group (order by counter_defno_i) absolute_due
                  from (
                         select case 
                                    when code = 'C' then to_char(trunc(due_at, 0))
                                    when code = 'H' then to_char(trunc(due_at/60, 0)) || ':' || to_char(round(((due_at/60) - trunc(due_at/60,0))*60, 0))
                                  else to_char(trunc(due_at,0))
                                 end 
                               ||
                               case 
                                   when code = 'C' then 'FC'
                                   when code = 'H' then 'FH'
                                 else code
                               end absolute_due
                            , cd.counter_defno_i
                            , event_perfno_i, due_at
                       from wo_transfer t
                       join wo_transfer_dimension d on t.event_transferno_i = d.event_transferno_i and is_last_transfer = 'Y'
                       join counter c on d.counterno_i = c.counterno_i
                       join counter_template ct on c.counter_templateno_i = ct.counter_templateno_i
                       join counter_definition cd on ct.counter_defno_i = cd.counter_defno_i and code <> 'D'
                      ) x
                 group by event_perfno_i
               ) x on h.event_perfno_i = x.event_perfno_i
   where h.workorderno_display is not null
     and h.event_type not in ('Q','T','J')
     and h.type in ('P','M','C','S')
     and f.expected_date >= trunc(sysdate - 7 - to_date('31.12.1971', 'dd.mm.yyyy'))
     and f.expected_date <= trunc(sysdate + 1 - to_date('31.12.1971', 'dd.mm.yyyy'))
     and f.event_type in ('W','R','V','C','D','T','Q','CR','DR','TR','QR')
     and (h.mel_code in ('A','B','C','D','L') or h.hil = 'Y')
     and a.status <> '9' /* только активные самолёты */
     and h.state = 'O'
     and t.is_last_transfer = 'Y'
     and a.ac_typ = 'A32S'
     and h.event_perfno_i in ('@98_MEL_Limit_group_wo.h.event_perfno_i@')
     order by expected_date asc
