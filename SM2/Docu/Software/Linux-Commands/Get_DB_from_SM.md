
#### Download DB via SCP command

(insert correct SM IP adress into xxx)

	scp pi@192.168.xxx.xxx:/var/sun2plug.db sun2plug.db   

#### Split DB into specific Time period

connect to SM via ssh

	sudo systemctl stop invertercontrol.service
	sudo python3 db_split.py /var/sun2plug.db YYYY-MM-DD YYYY-MM-DD

disconnect from SM

	scp pi@192.168.0.yyy:/var/output_YYYY-MM-DD_YYYY-MM-DD.db sun2plug_xxx.db   

or

	scp pi@192.168.xxx.xxx:/usr/lib/sun2plug/tests/output_YYYY-MM-DD_YYYY-MM-DD.db sun2plug_xxx.db   
