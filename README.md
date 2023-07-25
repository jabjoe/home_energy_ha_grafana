Quick write up
==============

This is just my hacking home energy Grafana setup built on Home Assistant.

It's not big or clever, but it might save someone time looking at this.

So all I've done is set Home Assistant to use MySQL.
Then I've added two extra tables in the database.

One for linking the metadata to a decent name (because HA uses a mix of DB and YAML) to display with Grafana.

The other is for day/night rates costing.


The first table is just:

    create table joe_name_links
    (
     name varchar(32),
     metadata_id int
    );

You add to it with say:

    insert into joe_name_links (name, metadata_id) VALUES('Home Power', 71); 
    insert into joe_name_links (name, metadata_id) VALUES('EV Charger Power', 34); 
    insert into joe_name_links (name, metadata_id) VALUES('Media Corner', 22); 
    insert into joe_name_links (name, metadata_id) VALUES('Drier', 3);
    insert into joe_name_links (name, metadata_id) VALUES('Internet', 14);


The second is complicated:


    create table joe_rate_links
    (
     id INT NOT NULL AUTO_INCREMENT,
     name varchar(32) NOT NULL,
     rate float NOT NULL,
     start_time TIME NOT NULL,
     end_time TIME NOT NULL,
     valid_from_ts BIGINT NOT NULL,
     valid_to_ts BIGINT,
     PRIMARY KEY (id)
    );

Basically the rate has a name, a time of day start and end, then valid to / from time stamps. This way you can support changing tarrif. 

So supposing you changed tarrif on 2023-07-12, and wanted the both in you DB so you could see the changing.

First you insert the old tarrif from UNIX origin (0) to the switch over date.

    insert into joe_rate_links (name, rate, start_time, end_time, valid_from_ts, valid_to_ts) VALUES("Day", 0.43, '00:00:00', '00:29:59', 0, UNIX_TIMESTAMP(date('2023-07-12')));
    insert into joe_rate_links (name, rate, start_time, end_time, valid_from_ts, valid_to_ts) VALUES("Night", 0.40, '00:30:00', '03:59:59', 0, UNIX_TIMESTAMP(date('2023-07-12')));
    insert into joe_rate_links (name, rate, start_time, end_time, valid_from_ts, valid_to_ts) VALUES("Day", 0.43, '04:00:00', '23:59:59', 0, UNIX_TIMESTAMP(date('2023-07-12')));

Second you'd insert the new tarrif from the switch over data to some distant date UNIX timestamp.

    insert into joe_rate_links (name, rate, start_time, end_time, valid_from_ts, valid_to_ts) VALUES("Day", 0.2961,'00:00:00','00:30:00', UNIX_TIMESTAMP(date('2023-07-12'), 32521103452);
    insert into joe_rate_links (name, rate, start_time, end_time, valid_from_ts, valid_to_ts) VALUES("Night", 0.2961, '00:30:00', '04:00:00', UNIX_TIMESTAMP(date('2023-07-12'), 32521103452);
    insert into joe_rate_links (name, rate, start_time, end_time, valid_from_ts, valid_to_ts) VALUES("Day", 0.2961, '04:00:00', '23:59:59', UNIX_TIMESTAMP(date('2023-07-12'), 32521103452);


