Installation of telegraf, influx and chronograf should be completed before
trying to configure logparser for exim.

You should have some nice CPU-graphs in chronograf before turning to exim.

= On Debian
telegraf won't have permission to read /var/log/exim4/mainlog.
There are several options to fix it, like adding the user "telegraf" to
group "adm".

Next, copy conf/exim.conf to /etc/telegraf/telegraf.d

Finally restart telegraf

	cp conf/exim.conf /etc/telegraf/telegraf.d/
        adduser telegraf adm
	systemctl restart telegraf.service

Installation is basically complete.


Verify using "influx":

	root@server ~ # influx
	Connected to http://localhost:8086 version 1.3.7
	InfluxDB shell version: 1.3.7
	> use telegraf
	Using database telegraf
	> select count(*) from exim_mainlog;
	name: exim_mainlog
	time count_eximid count_localpart count_rejectedhostname count_rejectedip count_size
	---- ------------ --------------- ---------------------- ---------------- ----------
	0    10           5               31                     31               5
	> 

Use "chronograf", create a new dashboad and a graph like
SELECT count("eximid") AS "count_eximid" FROM "telegraf"."autogen"."exim_mainlog" WHERE time > :dashboardTime: GROUP BY :interval:, "flag" FILL(null)

or 
SELECT count("rejectedip") AS "count_ip" FROM "telegraf"."autogen"."exim_mainlog" WHERE time > :dashboardTime: GROUP BY :interval:, "reason" FILL(null)


