Hi Guys.
We are going to discuss about installing the Multi Router Traffic Grapher (MRTG) Tool along with Round Robin Database (RRD) Tool on CentOS.
Note: The following version were used during the installation.

Operating System: CentOS 4 (Final), Architecture X86_64
MRTG: mrtg-2.17.4
RRD Tool: rrdtool-1.4.9

Certain commands need to be changed according to the versions (specifically the path of MRTG &amp; RRD Tools).

```bash
[root@localhos]# yum install -y gcc gd gd-devel perl libpng libxml2-devel cairo-devel glib2-devel pango-devel perl-devel perl-CGI net-snmp net-snmp-utils
```
Create a backup of the original snmpd.conf file.
```bash
[root@localhost]# mv /etc/snmp/snmpd.conf /etc/snmp/snmpd.conf.orig
```
Edit the snmpd.conf file
```bash
[root@localhost]# vi /etc/snmp/snmpd.conf
```
Replace the contents of the snmpd.conf file with the below lines.
```bash
com2sec local localhost public
com2sec mynetwork 192.168.1.0 public
group MyRWGroup v1 local
group MyRWGroup v2c local
group MyRWGroup usm local
group MyROGroup v1 mynetwork
group MyROGroup v2c mynetwork
group MyROGroup usm mynetwork
view all included .1 80
access MyROGroup "" any noauth exact all none none
access MyRWGroup "" any noauth exact all all none
syslocation New Delhi, INDIA
syscontact FooBar &lt;foor.bar@foobar.com&gt;
```
Note : the above file parameters can be modified as per requirement.
Restart the SNMP services.
```bash
[root@localhost]# /etc/init.d/snmpd restart
```
MRTG &amp; RRD Installation.
```bash
[root@localhost]# cd /tmp
[root@localhost]# wget http://oss.oetiker.ch/mrtg/pub/mrtg.tar.gz
[root@localhost]# tar zfx mrtg.tar.gz
[root@localhost]# cd mrtg-2.17.4
[root@localhost]# ./configure --prefix=/usr/local/mrtg2
[root@localhost]# make
[root@localhost]# make install
[root@localhost]# cd ..
[root@localhost]# wget http://oss.oetiker.ch/rrdtool/pub/rrdtool.tar.gz
[root@localhost]# tar zfx rrdtool.tar.gz
[root@localhost]# cd rrdtool-1.4.9/
[root@localhost]# ./configure --prefix=/usr/local/rrdtool
[root@localhost]# make
[root@localhost]# make install
[root@localhost]# mkdir /var/www/mrtg
[root@localhost]# mkdir/home/mrtg
[root@localhost]# /usr/local/mrtg2/bin/cfgmaker --global "WorkDir: /var/www/mrtg" --global "Options[_]: growright, bits" --global "RunAsDaemon: Yes" --global "LogFormat: rrdtool" --global "PathAdd: /usr/local/rrdtool/bin" --global "LibAdd: /usr/local/rrdtool/lib/perl/5.10.1" -ifref=ip --output /home/mrtg/mrtg.cfg public@localhost
[root@localhost]# env LANG=C /usr/local/mrtg2/bin/mrtg /home/mrtg/mrtg.cfg
```
Create a cron job for MRTG to generate data at every 5 mins.
```bash
[root@localhost]# vi /etc/cron.d/mrtg
```
Add the below lines to the MRTG cron file.
```bash
*/5 * * * * root env LANG=C /usr/local/mrtg2/bin/mrtg /home/mrtg/mrtg.cfg --logging /var/log/mrtg.log
```
Add the cron daemon to the server start-up.
```bash
[root@localhost]# chkconfig crond on
```
Now lets create the index.html file.
```bash
[root@localhost]# /usr/local/mrtg2/bin/indexmaker --output=/var/www/mrtg/index.html /home/mrtg/mrtg.cfg
```
Configure the Apache to handle MRTG. Create a mrtg.conf file in /etc/httpd/conf.d/
```bash
[root@localhost]# vi /etc/httpd/conf.d/mrtg.conf
```
Add the following lines to the mrtg.conf file
```bash
# This configuration file maps the mrtg output (generated daily)
# into the URL space. By default these results are only accessible
# from the local host.
#
Alias /mrtg /var/www/mrtg
&lt;Location /mrtg&gt;
Order deny,allow
#Deny from all
Allow from 127.0.0.1
Allow from ::1
# Allow from .example.com
Allow from all
&lt;/Location&gt;
```
Now add the httpd service to the server start-up &amp; restart the httpd service.
```bash
[root@localhost]# chkconfig httpd on
[root@localhost]# service httpd restart
```
Implementing CGI, Create a cgi file in the /var/www/cgi-bin.
```bash
[root@localhost]# vi /var/www/cgi-bin/14all.cgi
```
Now add the below lines to the CGI file /var/www/cgi-bin/14all.cgi
```bash
#!/usr/bin/perl -w
#
# 14all.cgi
#
# create html pages and graphics with rrdtool for mrtg + rrdtool
#
# (c) 1999-2002 bawidama@users.sourceforge.net
#
# use freely, but: NO WARRANTY - USE AT YOUR OWN RISK!
# license: GPL
# if MRTG_lib.pm (from mrtg) is not in the module search path (@INC)
# uncomment the following line and change the path appropriatly:
#use lib qw(/usr/local/mrtg-2/lib/mrtg2);
use lib qw(/usr/local/mrtg2/lib/mrtg2);
# if RRDs (rrdtool perl module) is not in the module search path (@INC)
# uncomment the following line and change the path appropriatly
# or use a LibAdd: setting in the config file
#use lib qw(/usr/local/rrdtool-1.0.38/lib/perl);
# RCS History - removed as it's available on the web
my $rcsid = '$Id: 14all.cgi,v 2.25 2003/01/05 18:39:51 bawidama Exp $';
my $subversion = (split(/ /,$rcsid))[2];
$subversion =~ m/^\d+\.(\d+)$/;
my $version = "14all.cgi 1.1p$1";
# my $DEBUG = 0; has gone - use config "14all*GraphErrorsToBrowser: 1"
use strict;
use CGI;
BEGIN { eval { require CGI::Carp; import CGI::Carp qw/fatalsToBrowser/ }
 if $^O !~ m/Win/i };
use MRTG_lib "2.090003";
sub main();
sub set_graph_params($$$$);
sub show_graph($$$);
sub show_log($$$);
sub show_dir($$$);
sub print_error($@);
sub intmax(@);
sub yesorno($);
sub get_graph_params($$$);
sub getdirwriteable($$);
sub getpngsize($);
sub errorpng();
sub gettextpic($);
sub log_rrdtool_call($$@);
# global, static vars
#$cfgfile = 'mrtg.cfg';
#$cfgfiledir = '/home/mrtg';
my ($cfgfile, $cfgfiledir);
my (@author, @style);
### where the mrtg.cfg file is
# anywhere in the filespace
$cfgfile = '/home/mrtg/mrtg.cfg';
# relative to the script
$cfgfile = 'mrtg.cfg';
# use this so 14all.cgi gets the cfgfile name from the script name
# (14all.cgi -&gt; 14all.cfg)
$cfgfile = 'mrtg.cfg';
# if you want to store your config files in a different place than your cgis:
$cfgfiledir = '/home/mrtg';
### customize the html pages
@author = ( -author =&gt; 'bawidama@users.sourceforge.net');
# one possibility to enable stylesheets (second is to use "AddHead[_]:..." in mrtg.cfg)
#@style = ( -style =&gt; { -src =&gt; 'general.css' });
###
# static data
my @headeropts = (@author, @style);
my %myrules = (
 '14all*errorpic' =&gt;
 [sub{$_[0] &amp;&amp; (-r $_[0] )}, sub{"14all*ErrorPic '$_[0]' not found/readable"}],
 '14all*grapherrorstobrowser' =&gt; [sub{1}, sub{"Internal Error"}],
 '14all*columns' =&gt;
 [sub{int($_[0]) &gt;= 1}, sub{"14all*Columns must be at least 1 (is '$_[0]')"}],
 '14all*rrdtoollog' =&gt;
 [sub{$_[0] &amp;&amp; (!-d $_[0] )}, sub{"14all*RRDToolLog must be a writable file"}],
 '14all*background' =&gt;
 [sub{$_[0] =~ /^#[0-9a-f]{6}$/i}, sub{"14all*background colour not in form '#xxxxxx', x in [a-f0-9]"}],
 '14all*logarithmic[]' =&gt; [sub{1}, sub{"Internal Error"}],
 '14all*graphtotal[]' =&gt; [sub{1}, sub{"Internal Error"}],
 '14all*dontshowindexgraph[]' =&gt; [sub{1}, sub{"Internal Error"}],
 '14all*indexgraph[]' =&gt; [sub{1}, sub{"Internal Error"}],
 '14all*indexgraphsize[]' =&gt;
 [sub{$_[0] =~ m/^\d+[,\s]+\d+$/o}, sub{"14all*indexgraphsize: need two numbers"}],
 '14all*maxrules[]' =&gt; [sub{1}, sub{"Internal Error"}],
 '14all*stackgraph[]' =&gt; [sub{1}, sub{"Internal Error"}],
);
my %graphparams = (
 'daily' =&gt; ['-2000m', 'now', 300],
 'weekly' =&gt; ['-12000m','now', 1800],
 'monthly' =&gt; ['-800h', 'now', 7200],
 'yearly' =&gt; ['-400d', 'now', 86400],
 'daily.s' =&gt; ['-1250m', 'now', 300],
);
# the footer we print on every page
my $footer_template = &lt;&lt;"EOT";
&lt;TABLE BORDER=0 CELLSPACING=0 CELLPADDING=0&gt;
 &lt;TR&gt;
 &lt;TD WIDTH=63&gt;&lt;A ALT="MRTG"
 HREF="http://ee-staff.ethz.ch/~oetiker/webtools/mrtg/mrtg.html"&gt;&lt;IMG
 BORDER=0 SRC="##ICONDIR##mrtg-l.gif"&gt;&lt;/A&gt;&lt;/TD&gt;
 &lt;TD WIDTH=25&gt;&lt;A ALT=""
 HREF="http://ee-staff.ethz.ch/~oetiker/webtools/mrtg/mrtg.html"&gt;&lt;IMG
 BORDER=0 SRC="##ICONDIR##mrtg-m.gif"&gt;&lt;/A&gt;&lt;/TD&gt;
 &lt;TD WIDTH=388&gt;&lt;A ALT=""
 HREF="http://ee-staff.ethz.ch/~oetiker/webtools/mrtg/mrtg.html"&gt;&lt;IMG
 BORDER=0 SRC="##ICONDIR##mrtg-r.gif"&gt;&lt;/A&gt;&lt;/TD&gt;
 &lt;/TR&gt;
&lt;/TABLE&gt;
&lt;TABLE BORDER=0 CELLSPACING=0 CELLPADDING=0&gt;
 &lt;TR VALIGN=top&gt;
 &lt;TD WIDTH=88 ALIGN=RIGHT&gt;&lt;FONT FACE="Arial,Helvetica" SIZE=2&gt;Version ${MRTG_lib::VERSION}&lt;/FONT&gt;&lt;/TD&gt;
 &lt;TD WIDTH=388 ALIGN=RIGHT&gt;&lt;FONT FACE="Arial,Helvetica" SIZE=2&gt;
 &lt;A HREF="http://ee-staff.ethz.ch/~oetiker/"&gt;Tobias Oetiker&lt;/A&gt;
 &lt;A HREF="mailto:oetiker\@ee.ethz.ch"&gt;&amp;lt;oetiker\@ee.ethz.ch&amp;gt;&lt;/A&gt; and
 &lt;A HREF="http://www.bungi.com"&gt;Dave&amp;nbsp;Rand&lt;/A&gt;&amp;nbsp;
 &lt;A HREF="mailto:dlr\@bungi.com"&gt;&amp;lt;dlr\@bungi.com&amp;gt;&lt;/A&gt;
 &lt;TR VALIGN=top&gt;
 &lt;TD WIDTH=88 ALIGN=RIGHT&gt;&lt;FONT FACE="Arial,Helvetica" SIZE=2&gt;&lt;a
 href="http://my14all.sourceforge.net/"&gt;$version&lt;/a&gt;&lt;/FONT&gt;&lt;/TD&gt;
 &lt;TD WIDTH=388 ALIGN=RIGHT&gt;&lt;FONT FACE="Arial,Helvetica" SIZE=2&gt;
 Rainer&amp;nbsp;Bawidamann&amp;nbsp;
 &lt;A HREF="mailto:bawidama\@users.sourceforge.net"&gt;&amp;lt;bawidama\@users.sourceforge.net&amp;gt;&lt;/A&gt;&lt;/FONT&gt;
 &lt;/TD&gt;
&lt;/TR&gt;
&lt;/TABLE&gt;
EOT
# global vars, persistent for speecyCGI/mod_perl
%my14all::config_cache = ();
$my14all::meurl = '';
# run main
main();
exit(0);
# end.
sub main() {
 if (!$cfgfile &amp;&amp; $#ARGV == 0 &amp;&amp; $ARGV[0] !~ m/=/ &amp;&amp; !exists($ENV{QUERY_STRING})) {
 $cfgfile = shift @ARGV;
 }
 my %setup;
 # initialize CGI
 my $q = new CGI;
 $setup{q} = $q;
 # change for mrtg-2.9.*
 # look for the config file
 $my14all::meurl = $q-&gt;url(-relative =&gt; 1);
 ensureSL(\$cfgfiledir);
 my $l_cfgfile; # don't change what is set above in the CGI
 if (defined $q-&gt;param('cfg')) {
 $l_cfgfile = $q-&gt;param('cfg');
 # security fix: don't allow ./ in the config file name
 print_error($q, "Illegal characters in cfg param: ./")
 if $l_cfgfile =~ m'(^/)|(\./)';
 $l_cfgfile = $cfgfiledir.$l_cfgfile unless -r $l_cfgfile;
 print_error($q, "Cannot find the given config file: \&lt;tt&gt;$l_cfgfile\&lt;/tt&gt;")
 unless -r $l_cfgfile;
 } elsif (!$cfgfile) {
 $l_cfgfile = $my14all::meurl;
 $l_cfgfile =~ s|.*\Q$MRTG_lib::SL\E||;
 $l_cfgfile =~ s/\.(cgi|pl|perl)$/.cfg/;
 #$my14all::meurl =~ m{\Q$MRTG_lib::SL\E([^\Q$MRTG_lib::SL\E]*)\.(cgi|pl)$};
 #$l_cfgfile = $1 . '.cfg';
 $l_cfgfile = $cfgfiledir.$l_cfgfile unless -r $l_cfgfile;
 } else {
 $l_cfgfile = $cfgfile;
 }
 # now we have the name of the config file.
 # we might already have it with speedyCGI or mod_perl
 my $read_file = 1;
 my @cfg_stat = stat($l_cfgfile);
 if (exists $my14all::config_cache{config}{$l_cfgfile}{mtime} &amp;&amp;
 $cfg_stat[9] &lt;= $my14all::config_cache{config}{$l_cfgfile}{mtime})
 {
 $read_file = 0;
 }
 # read the config file
 if ($read_file) {
 # wipe old config
 delete $my14all::config_cache{config}{$l_cfgfile}
 if exists $my14all::config_cache{config}{$l_cfgfile};
 my (@sorted, %config, %targets);
 readcfg($l_cfgfile, \@sorted, \%config, \%targets, "14all", \%myrules);
 my @processed_targets;
 cfgcheck(\@sorted, \%config, \%targets, \@processed_targets);
 # set default refresh
 if (exists $config{refresh} &amp;&amp; yesorno($config{refresh})
 &amp;&amp; $config{refresh} !~ m/^\d*[1-9]\d*$/o) {
 $config{refresh} = $config{interval} * 60;
 }
 # store config in "cache"
 $my14all::config_cache{config}{$l_cfgfile}{sorted} = \@sorted;
 $my14all::config_cache{config}{$l_cfgfile}{config} = \%config;
 $my14all::config_cache{config}{$l_cfgfile}{targets} = \%targets;
 $my14all::config_cache{config}{$l_cfgfile}{processed_targets} = \@processed_targets;
 $my14all::config_cache{config}{$l_cfgfile}{mtime} = $cfg_stat[9];
 # add LibAdd from config for RRDs.pm
 if ($config{libadd}) {
 unshift @INC, $config{libadd};
 }
 # load the rrdtool perl extension
 eval "use RRDs 1.00038";
 }
 my $cfg = $my14all::config_cache{config}{$l_cfgfile};
 my $footer = $footer_template;
 # make sure icondir is set (should be done by mrtg)
 $cfg-&gt;{config}{icondir} ||= $cfg-&gt;{config}{workdir};
 # put icondir into footer
 $footer =~ s/##ICONDIR##/$cfg-&gt;{config}{icondir}/g;
 my $log = '_';
 if (defined $q-&gt;param('log')) {
 $log = $q-&gt;param('log');
 }
 # set timezone for rrdtool
 if ($cfg-&gt;{targets}{timezone}{$log}) {
 $ENV{"TZ"} = $cfg-&gt;{targets}{timezone}{$log};
 }
 ### the main switch
 # the modes:
 # if parameter "dir" is given show a list of the targets in this "directory"
 # elsif parameter "png" is given show a graphic for the target given w/ parameter "log"
 # elsif parameter "log" is given show the page for this target
 # else show a list of directories and of targets w/o directory
 # parameter "cfg" can hold the name of the config file to use
 if (defined $q-&gt;param('dir')) {
 show_dir($cfg, $q, $q-&gt;param('dir'));
 } elsif (defined $q-&gt;param('png')) {
 show_graph($cfg, $q, $q-&gt;param('png'));
 exit(0); # don't show (html) footer
 } elsif (defined $q-&gt;param('log')) {
 show_log($cfg, $q, $q-&gt;param('log'));
 } else {
 show_dir($cfg, $q, '');
 }
 if (!$cfg-&gt;{targets}{options}{nobanner}{$log}) {
 print $footer;
 }
 print $q-&gt;end_html();
 exit(0);
}
sub set_graph_params($$$$) {
 # calculate graph params and store in config
 my ($cfg, $log, $png, $small) = @_;
 unless (exists $cfg-&gt;{targets}{target}{$log}) {
 return "target '$log' unknown";
 }
 my ($start, $end, $maxage);
 my $graphparams = $cfg-&gt;{targets}{"graph*$png"}{$log};
 if ($graphparams) {
 ($start, $end, $maxage) = split(/[,\s]+/, $graphparams, 3);
 }
 unless ($start &amp;&amp; $end &amp;&amp; $maxage) {
 unless (exists $graphparams{$png}) {
 return "CGI call error: graph '$png' unknown";
 }
 ($start, $end, $maxage) = @{$graphparams{$png}};
 }
 my ($xs, $ys);
 if ($small) {
 ($xs, $ys) = (250, 100);
 ($xs, $ys) = ($cfg-&gt;{targets}{'14all*indexgraphsize'}{$log} =~ m/(\d+)[,\s]+(\d+)/)
 if $cfg-&gt;{targets}{'14all*indexgraphsize'}{$log};
 } else {
 ($xs, $ys) = ($cfg-&gt;{targets}{xsize}{$log}, $cfg-&gt;{targets}{ysize}{$log});
 }
 unless ($xs &amp;&amp; $ys) {
 return "cannot get image sizes for graph $png / target $log";
 }
 my $rrd = $cfg-&gt;{config}{logdir}.$cfg-&gt;{targets}{directory}{$log} . $log . '.rrd';
 # escape ':' and '\' with \ in $rrd
 # (rrdtool replaces '\:' by ':' and '\\' by '\')
 $rrd =~ s/([:\\])/\\$1/g;
 my $pngdir = getdirwriteable($cfg-&gt;{config}{imagedir}, $cfg-&gt;{targets}{directory}{$log});
 $png .= '-i' if $small;
 my $pngfile = "${pngdir}${log}-${png}.png";
 # build the rrd command line: set the starttime and the graphics format (PNG)
 my @args = ($pngfile, '-s', $start, '-e', $end, '-a', 'PNG');
 # if it's not a small picture set the legends
 my ($l1,$l2,$l3,$l4,$li,$lo) = ('','','','','','');
 my ($ri, $ro) = ('','');
 # set the size of the graph
 push @args, '-w', $xs, '-h', $ys;
 # set scale factor and legend defaults
 my $per_unit = 'Second';
 my $persec = 'Bytes';
 my $factor = 1; # should we scale the values?
 if ($cfg-&gt;{targets}{options}{perminute}{$log}) {
 $factor = 60; # perminute -&gt; 60x
 $per_unit = 'Minute';
 } elsif ($cfg-&gt;{targets}{options}{perhour}{$log}) {
 $factor = 3600; # perhour -&gt; 3600x
 $per_unit = 'Hour';
 }
 if ($cfg-&gt;{targets}{options}{bits}{$log}) {
 $factor *= 8; # bits instead of bytes -&gt; 8x
 $persec = 'Bits';
 }
 # let the user give an arbitrary factor:
 if ($cfg-&gt;{targets}{factor}{$log} and
 $cfg-&gt;{targets}{factor}{$log} =~ m/^[-+]?\d+(.\d+)?([eE][+-]?\d+)?$/)
 {
 $factor *= 0+$cfg-&gt;{targets}{factor}{$log};
 }
 # check if only one value should be graphed (set vars for fast access)
 my $noi = $cfg-&gt;{targets}{options}{noi}{$log};
 my $noo = $cfg-&gt;{targets}{options}{noo}{$log};
 # prepare the legends
 if (!$small &amp;&amp; !$cfg-&gt;{targets}{options}{nolegend}{$log}) {
 foreach (qw/legend1 legend2 legend3 legend4 legendi legendo legendy shortlegend/) {
 if ($cfg-&gt;{targets}{$_}{$log}) {
 $cfg-&gt;{targets}{$_}{$log} =~ s'&amp;nbsp;' 'go; #'
 $cfg-&gt;{targets}{$_}{$log} =~ s/%/%%/go;
 }
 }
 if ($cfg-&gt;{targets}{ylegend}{$log}) {
 push @args, '-v', $cfg-&gt;{targets}{ylegend}{$log};
 } else {
 push @args, '-v', "$persec per $per_unit";
 }
 if ($cfg-&gt;{targets}{legend1}{$log}) {
 $l1 = ":".$cfg-&gt;{targets}{legend1}{$log}."\\l"; }
 else {
 $l1 = ":Incoming Traffic in $persec per $per_unit\\l"; }
 if ($cfg-&gt;{targets}{legend2}{$log}) {
 $l2 = ":".$cfg-&gt;{targets}{legend2}{$log}."\\l"; }
 else {
 $l2 = ":Outgoing Traffic in $persec per $per_unit\\l"; }
 if ($cfg-&gt;{targets}{legend3}{$log}) {
 $l3 = ":".$cfg-&gt;{targets}{legend3}{$log}."\\l"; }
 else {
 $l3 = ":Maximal 5 Minute Incoming Traffic\\l"; }
 if ($cfg-&gt;{targets}{legend4}{$log}) {
 $l4 = ":".$cfg-&gt;{targets}{legend4}{$log}."\\l"; }
 else {
 $l4 = ":Maximal 5 Minute Outgoing Traffic\\l"; }
 if (exists $cfg-&gt;{targets}{legendi}{$log}) {
 $li = $cfg-&gt;{targets}{legendi}{$log}; }
 else { $li = "In: "; }
 $li =~ s':'\\:'; # ' quote :
 if (exists $cfg-&gt;{targets}{legendo}{$log}) {
 $lo = $cfg-&gt;{targets}{legendo}{$log}; }
 else { $lo = "Out:"; }
 $lo =~ s':'\\:'; # ' quote :
 if ($cfg-&gt;{targets}{options}{integer}{$log}) {
 $li .= ' %9.0lf' if $li;
 $lo .= ' %9.0lf' if $lo;
 $ri = '%3.0lf%%';
 $ro = '%3.0lf%%';
 } else {
 $li .= ' %8.3lf' if $li;
 $lo .= ' %8.3lf' if $lo;
 $ri = '%6.2lf%%';
 $ro = '%6.2lf%%';
 }
 if (!exists($cfg-&gt;{targets}{kmg}{$log}) || $cfg-&gt;{targets}{kmg}{$log}) {
 $li .= ' %s' if $li;
 $lo .= ' %s' if $lo;
 if ($cfg-&gt;{targets}{kilo}{$log}) {
 push @args, '-b', $cfg-&gt;{targets}{kilo}{$log};
 }
 if ($cfg-&gt;{targets}{shortlegend}{$log}) {
 $li .= $cfg-&gt;{targets}{shortlegend}{$log} if $li;
 $lo .= $cfg-&gt;{targets}{shortlegend}{$log} if $lo;
 }
 }
 }
 my $pngchar = substr($png,0,1);
 if ($pngchar and $cfg-&gt;{targets}{unscaled}{$log} and
 $cfg-&gt;{targets}{unscaled}{$log} =~ m/$pngchar/) {
 my $max = intmax($cfg-&gt;{targets}{maxbytes}{$log},
 $cfg-&gt;{targets}{maxbytes1}{$log},
 $cfg-&gt;{targets}{maxbytes2}{$log},
 $cfg-&gt;{targets}{absmax}{$log});
 $max *= $factor;
 push @args, '-l', 0, '-u', $max, '-r';
 } elsif (yesorno($cfg-&gt;{targets}{'14all*logarithmic'}{$log})) {
 push @args, '-o';
 }
 push @args,'--alt-y-mrtg','--lazy','-c','MGRID#ee0000','-c','GRID#000000';
 # contributed by Henry Chen: option to stack the two values
 # set the mode of the second line from config
 my $line2 = 'LINE1:';
 if (yesorno($cfg-&gt;{targets}{'14all*stackgraph'}{$log})) {
 $line2 = 'STACK:';
 }
 # now build the graph calculation commands
 # ds0/ds1 hold the normal data sources to graph/gprint
 my ($ds0, $ds1) = ('in', 'out');
 push @args, "DEF:$ds0=$rrd:ds0:AVERAGE", "DEF:$ds1=$rrd:ds1:AVERAGE";
 if (defined $cfg-&gt;{targets}{options}{unknaszero}{$log}) {
 push @args, "CDEF:uin=$ds0,UN,0,$ds0,IF",
 "CDEF:uout=$ds1,UN,0,$ds1,IF";
 ($ds0, $ds1) = ('uin', 'uout');
 }
 if ($factor != 1) {
 # scale the values. we need a CDEF for this
 push @args, "CDEF:fin=$ds0,$factor,*","CDEF:fout=$ds1,$factor,*";
 ($ds0, $ds1) = ('fin', 'fout');
 }
 my $maximum0 = $cfg-&gt;{targets}{maxbytes1}{$log} || $cfg-&gt;{targets}{maxbytes}{$log};
 my $maximum1 = $cfg-&gt;{targets}{maxbytes2}{$log} || $cfg-&gt;{targets}{maxbytes}{$log};
 $maximum0 = 1 unless $maximum0;
 $maximum1 = 1 unless $maximum1;
 # ps0/ps1 hold the percentage data source for gprint
 my ($ps0, $ps1) = ('pin', 'pout');
 push @args, "CDEF:pin=$ds0,$maximum0,/,100,*,$factor,/",
 "CDEF:pout=$ds1,$maximum1,/,100,*,$factor,/";
 if (yesorno($cfg-&gt;{targets}{'14all*graphtotal'}{$log})
 &amp;&amp; $small) {
 push @args, "CDEF:total=$ds0,$ds1,+", "LINE1:total#ffa050:Total AVG\\l";
 }
 # now for the peak graphs / maximum values
 # mx0/mx1 hold the maximum data source for graph/gprint
 my ($mx0, $mx1) = ($ds0, $ds1);
 # px0/px1 hold the maximum pecentage data source for gprint
 my ($px0, $px1) = ($ps0, $ps1);
 if (!$small) {
 # the defs for the maximum values: for the legend ('MAX') and probabely
 # for the 'withpeak' graphs
 push @args, "DEF:min=$rrd:ds0:MAX", "DEF:mout=$rrd:ds1:MAX";
 ($mx0, $mx1) = ('min', 'mout');
 if (defined $cfg-&gt;{targets}{options}{unknaszero}{$log}) {
 push @args, "CDEF:umin=$mx0,UN,0,$mx0,IF",
 "CDEF:umout=$mx1,UN,0,$mx1,IF";
 ($mx0, $mx1) = ('umin', 'umout');
 }
 if ($factor != 1) {
 # scale the values. we need a CDEF for this
 push @args, "CDEF:fmin=$mx0,$factor,*","CDEF:fmout=$mx1,$factor,*";
 ($mx0, $mx1) = ('fmin', 'fmout');
 }
 # draw peak lines if configured
 if ($cfg-&gt;{targets}{withpeak}{$log} &amp;&amp;
 substr($png,0,1) =~ /[$cfg-&gt;{targets}{withpeak}{$log} ]/) {
 push @args, "AREA:".$mx0.$cfg-&gt;{targets}{rgb3}{$log}.$l3 unless $noi;
 push @args, $line2.$mx1.$cfg-&gt;{targets}{rgb4}{$log}.$l4 unless $noo;
 if (yesorno($cfg-&gt;{targets}{'14all*graphtotal'}{$log})) {
 push @args, "CDEF:mtotal=$mx0,$mx1,+", "LINE1:mtotal#ff5050:Total MAX\\l";
 }
 }
 push @args, "CDEF:pmin=$mx0,$maximum0,/,100,*,$factor,/",
 "CDEF:pmout=$mx1,$maximum1,/,100,*,$factor,/";
 ($px0, $px1) = ('pmin', 'pmout');
 }
 # the commands to draw the values
 my (@hr1, @hr2);
 if (!$small &amp;&amp; yesorno($cfg-&gt;{targets}{'14all*maxrules'}{$log})) {
 chop $l1; chop $l1; chop $l2; chop $l2;
 @hr1 = (sprintf("HRULE:%d#ff4400: MaxBytes1\\l",$maximum0*$factor));
 @hr2 = (sprintf("HRULE:%d#aa0000: MaxBytes2\\l",$maximum1*$factor));
 }
 push @args, "AREA:".$ds0.$cfg-&gt;{targets}{rgb1}{$log}.$l1, @hr1 unless $noi;
 push @args, $line2.$ds1.$cfg-&gt;{targets}{rgb2}{$log}.$l2, @hr2 unless $noo;
 if (!$small &amp;&amp; !$cfg-&gt;{targets}{options}{nolegend}{$log}) {
 # print the legends
 # order matters so noi/noo makes this ugly
 if ($cfg-&gt;{targets}{options}{nopercent}{$log}) {
 push @args, "GPRINT:$mx0:MAX:Maximal $li" if $li &amp;&amp; !$noi;
 push @args, "GPRINT:$mx1:MAX:Maximal $lo\\l" if $lo &amp;&amp; !$noo;
 push @args, "GPRINT:$ds0:AVERAGE:Average $li" if $li &amp;&amp; !$noi;
 push @args, "GPRINT:$ds1:AVERAGE:Average $lo\\l" if $lo &amp;&amp; !$noo;
 push @args, "GPRINT:$ds0:LAST:Current $li" if $li &amp;&amp; !$noi;
 push @args, "GPRINT:$ds1:LAST:Current $lo\\l" if $lo &amp;&amp; !$noo;
 } else {
 push @args,
 "GPRINT:$mx0:MAX:Maximal $li",
 "GPRINT:$px0:MAX:($ri)" if $li &amp;&amp; !$noi;
 push @args,
 "GPRINT:$mx1:MAX:Maximal $lo",
 "GPRINT:$px1:MAX:($ro)\\l" if $lo &amp;&amp; !$noo;
 push @args,
 "GPRINT:$ds0:AVERAGE:Average $li",
 "GPRINT:$ps0:AVERAGE:($ri)" if $li &amp;&amp; !$noi;
 push @args,
 "GPRINT:$ds1:AVERAGE:Average $lo",
 "GPRINT:$ps1:AVERAGE:($ro)\\l" if $lo &amp;&amp; !$noo;
 push @args,
 "GPRINT:$ds0:LAST:Current $li",
 "GPRINT:$ps0:LAST:($ri)" if $li &amp;&amp; !$noi;
 push @args,
 "GPRINT:$ds1:LAST:Current $lo",
 "GPRINT:$ps1:LAST:($ro)\\l" if $lo &amp;&amp; !$noo;
 }
 }
 # store rrdtool arg list for speedCGI/mod_perl mode
 $cfg-&gt;{rrdargs}{$log}{$png}{args} = \@args;
 $cfg-&gt;{rrdargs}{$log}{$png}{pngdir} = $pngdir;
 $cfg-&gt;{rrdargs}{$log}{$png}{pngfile} = $pngfile;
 $cfg-&gt;{rrdargs}{$log}{$png}{maxage} = $maxage;
 return '';
}
sub show_graph($$$) {
 # send a graphic, create it if necessary
 my ($cfg, $q, $png) = @_;
 my $log = $q-&gt;param('log');
 my $small = $q-&gt;param('small') ? '-i' : '';
 my $errstr = '';
 if (!$log) {
 $errstr="CGI call error: missing param 'log'";
 goto ERROR;
 }
 unless (exists $cfg-&gt;{targets}{target}{$log}) {
 $errstr="target '$log' unknown";
 goto ERROR;
 }
 # fix a problem with indexmaker
 if ($small) {
 my %imaker = qw/day.s daily.s week.s weekly month.s monthly year.s yearly/;
 if (exists $imaker{$png}) {
 $png = $imaker{$png};
 }
 }
 my $argsref;
 if (!exists $cfg-&gt;{rrdargs}{$log}{"$png$small"}{args}) {
 $errstr = set_graph_params($cfg, $log, $png, $small);
 goto ERROR if $errstr;
 }
 $argsref = $cfg-&gt;{rrdargs}{$log}{"$png$small"}{args};
 # fire up rrdtool
 my ($a, $rrdx, $rrdy) = RRDs::graph(@$argsref);
 my $e = RRDs::error();
 log_rrdtool_call($cfg-&gt;{config}{'14all*rrdtoollog'},$e,'graph',$argsref);
 my $pngfile = $cfg-&gt;{rrdargs}{$log}{"$png$small"}{pngfile};
 if ($e) {
 my $pngdir = $cfg-&gt;{rrdargs}{$log}{"$png$small"}{pngdir};
 if (!-w $pngdir) {
 $errstr = "cannot write to graph dir $pngdir\nrrdtool error: $e";
 } elsif (-e $pngfile and !-w _) {
 $errstr = "cannot write $pngfile\nrrdtool error: $e";
 } elsif (-e $pngfile) {
 if (unlink($pngfile)) {
 # try rrdtool a second time
 ($a, $rrdx, $rrdy) = RRDs::graph(@$argsref);
 $e = RRDs::error();
 log_rrdtool_call($cfg-&gt;{config}{'14all*rrdtoollog'},$e,'graph',$argsref);
 $errstr = $e ? $errstr."\nrrdtool error from 2. call: $e" : '';
 } else {
 $errstr = "cannot delete file $pngfile: $!";
 }
 } else {
 $errstr = "cannot create graph\nrrdtool error: $e";
 }
 }
 unless ($errstr) {
 if (open(PNG, "&lt;$pngfile")) {
 my $maxage = $cfg-&gt;{rrdargs}{$log}{"$png$small"}{maxage};
 print $q-&gt;header(-type =&gt; "image/png", -expires =&gt; "+${maxage}s");
 binmode(PNG); binmode(STDOUT);
 while(read PNG, my $buf, 16384) { print STDOUT $buf; }
 close PNG;
 return;
 }
 $errstr = "cannot read graph file: $!";
 }
 ERROR:
 if (yesorno($cfg-&gt;{config}{'14all*grapherrorstobrowser'})) {
 my ($errpic, $format) = gettextpic($errstr);
 print $q-&gt;header(-type =&gt; $format, -expires =&gt; 'now');
 binmode(STDOUT);
 print $errpic;
 return;
 }
 $log ||= '_';
 if (defined $cfg-&gt;{targets}{options}{'14all*errorpic'}{$log} &amp;&amp;
 open(PNG, $cfg-&gt;{targets}{options}{'14all*errorpic'}{$log})) {
 print $q-&gt;header(-type =&gt; "image/png", -expires =&gt; 'now');
 binmode(PNG); binmode(STDOUT);
 while(read PNG, my $buf,16384) { print STDOUT $buf; }
 close PNG;
 return;
 }
 print $q-&gt;header(-type =&gt; "image/png", -expires =&gt; 'now');
 binmode(STDOUT);
 print pack("C*", errorpng());
}
sub show_log($$$) {
 # show the graphics for one target
 my ($cfg, $q, $log) = @_;
 print_error($q,"Target '$log' unknown") if (!exists $cfg-&gt;{targets}{target}{$log});
 my $title;
 # user defined title?
 if ($cfg-&gt;{targets}{title}{$log}) {
 $title = $cfg-&gt;{targets}{title}{$log};
 } else {
 $title = "MRTG/RRD - Target $log";
 }
 my @httphead;
 push @httphead, (-expires =&gt; '+' . int($cfg-&gt;{config}{interval}) . 'm');
 if (yesorno($cfg-&gt;{config}{refresh})) {
 push @httphead, (-refresh =&gt; $cfg-&gt;{config}{refresh});
 }
 my @htmlhead = (-title =&gt; $title, @headeropts,
 -bgcolor =&gt; $cfg-&gt;{targets}{background}{$log});
 if ($cfg-&gt;{targets}{addhead}{$log}) {
 push @htmlhead, (-head =&gt; $cfg-&gt;{targets}{addhead}{$log});
 }
 print $q-&gt;header(@httphead), $q-&gt;start_html(@htmlhead);
 # user defined header line? (should exist as mrtg requires it)
 print $cfg-&gt;{targets}{pagetop}{$log},"\n";
 my $rrd = $cfg-&gt;{config}{logdir}.$cfg-&gt;{targets}{directory}{$log} . $log . '.rrd';
 my $lasttime = RRDs::last($rrd);
 log_rrdtool_call($cfg-&gt;{config}{'14all*rrdtoollog'},'','last',$rrd);
 print $q-&gt;hr,
 "The statistics were last updated: ",$q-&gt;b(scalar(localtime($lasttime))),
 $q-&gt;hr if $lasttime;
 my $sup = $cfg-&gt;{targets}{suppress}{$log} || '';
 my $url = "$my14all::meurl?log=$log";
 my $tmpcfg = $q-&gt;param('cfg');
 $url .= "&amp;cfg=$tmpcfg" if defined $tmpcfg;
 $url .= "&amp;png";
 # the header lines and tags for the graphics
 my $pngdir = getdirwriteable($cfg-&gt;{config}{imagedir}, $cfg-&gt;{targets}{directory}{$log});
 if ($sup !~ /d/) {
 print $q-&gt;h2("'Daily' graph (5 Minute Average)"),"\n",
 $q-&gt;img({src =&gt; "$url=daily", alt =&gt; "daily-graph",
 getpngsize("$pngdir$log-daily.png")}
 ), "\n";
 }
 if ($sup !~ /w/) {
 print $q-&gt;h2("'Weekly' graph (30 Minute Average)"),"\n",
 $q-&gt;img({src =&gt; "$url=weekly", alt =&gt; "weekly-graph",
 getpngsize("$pngdir$log-weekly.png")}
 ), "\n";
 }
 if ($sup !~ /m/) {
 print $q-&gt;h2("'Monthly' graph (2 Hour Average)"),"\n",
 $q-&gt;img({src =&gt; "$url=monthly", alt =&gt; "monthly-graph",
 getpngsize("$pngdir$log-monthly.png")}
 ), "\n";
 }
 if ($sup !~ /y/) {
 print $q-&gt;h2("'Yearly' graph (1 Day Average)"),"\n",
 $q-&gt;img({src =&gt; "$url=yearly", alt =&gt; "yearly-graph",
 getpngsize("$pngdir$log-yearly.png")}
 ), "\n";
 }
 if ($cfg-&gt;{targets}{pagefoot}{$log}) {
 print $cfg-&gt;{targets}{pagefoot}{$log};
 }
}
sub show_dir($$$) {
 # if no parameter - show a list of directories and targets
 # without "Directory[...]" (aka root-targets)
 # else show a list of targets in the given directory
 my ($cfg, $q, $dir) = @_;
 my @httphead;
 push @httphead, (-expires =&gt;
 ($dir ? '+' . int($cfg-&gt;{config}{interval}) . 'm' : '+1d') );
 if (yesorno($cfg-&gt;{config}{refresh})) {
 push @httphead, (-refresh =&gt; $cfg-&gt;{config}{refresh});
 }
 push @headeropts, (-bgcolor =&gt; ($cfg-&gt;{config}{'14all*background'} || '#ffffff'));
 my @htmlhead = (-title =&gt;
 ($dir ? "MRTG/14all - Group $dir" : "MRTG/14all $version"),
 @headeropts);
 print $q-&gt;header(@httphead), $q-&gt;start_html(@htmlhead);
 my (@dirs, %dirs, @logs);
 # get the list of directories and "root"-targets
 # or get list of targets from given directory
 foreach my $tar (@{$cfg-&gt;{sorted}}) {
 next if $tar =~ m/^[_\$\^]$/; # pseudo targets
 if ($cfg-&gt;{targets}{directory}{$tar}) {
 if ($dir) {
 # show a specified dir. check this targets dir against specified
 if ($dir eq $cfg-&gt;{targets}{directory}{$tar}) {
 # target is from specified dir
 push @logs, $tar;
 }
 next;
 }
 next if exists $dirs{$cfg-&gt;{targets}{directory}{$tar}};
 $dirs{$cfg-&gt;{targets}{directory}{$tar}} = $tar;
 push @dirs, $cfg-&gt;{targets}{directory}{$tar};
 } elsif (!$dir) {
 # showing 'homepage', add this target
 push @logs, $tar;
 }
 }
 my $cfgstr = (defined $q-&gt;param('cfg') ? "&amp;cfg=".$q-&gt;param('cfg') : '');
 print $q-&gt;h1("Available Targets"),"\n";
 my $confcolumns = $cfg-&gt;{config}{'14all*columns'} || 2;
 if ($#dirs &gt; -1) {
 print $q-&gt;h2("Directories"),"\n\&lt;table width=100\%&gt;\n";
 my $column = 0;
 foreach my $tar (@dirs) {
 print '&lt;tr&gt;' if $column == 0;
 (my $link = $tar) =~ s/ /\+/g;
 chop $tar; # remove / for display (from ensureSL)
 print $q-&gt;td($q-&gt;a({href =&gt; "$my14all::meurl?dir=$link$cfgstr"},
 $tar)),"\n";
 $column++;
 if ($column &gt;= $confcolumns) {
 $column = 0;
 print '&lt;/tr&gt;';
 }
 }
 if ($column != 0 and $column &lt; $confcolumns) {
 print '&lt;td&gt;&amp;nbsp;&lt;/td&gt;' x ($confcolumns - $column),"\&lt;/tr&gt;\n";
 }
 print '&lt;/table&gt;&lt;hr&gt;';
 }
 my $pngdir = getdirwriteable($cfg-&gt;{config}{imagedir},$dir);
 if ($#logs &gt; -1) {
 print $q-&gt;h2("Targets"),"\n\&lt;table width=100\%&gt;\n";
 my $column = 0;
 foreach my $tar (@logs) {
 my $small = 0;
 unless (yesorno($cfg-&gt;{targets}{'14all*dontshowindexgraph'}{$tar})) {
 $small = $cfg-&gt;{targets}{'14all*indexgraph'}{$tar};
 $small = 'daily.s' unless $small;
 }
 next if $tar =~ m/^[\$\^_]$/; # _ is not a real target
 print '&lt;tr&gt;' if $column == 0;
 print '&lt;td&gt;',
 $q-&gt;p($q-&gt;a({href =&gt; "$my14all::meurl?log=$tar$cfgstr"}, $cfg-&gt;{targets}{title}{$tar}));
 print $q-&gt;a({href =&gt; "$my14all::meurl?log=$tar$cfgstr"},
 $q-&gt;img({src =&gt; "$my14all::meurl?log=$tar&amp;png=$small&amp;small=1$cfgstr",
 alt =&gt; "index-graph",
 getpngsize("$pngdir$tar-$small-i.png")}))
 if $small;
 print "\&lt;/td&gt;\n";
 $column++;
 if ($column &gt;= $confcolumns) {
 $column = 0;
 print '&lt;/tr&gt;';
 }
 }
 if ($column != 0 and $column &lt; $confcolumns) {
 print '&lt;td&gt;&amp;nbsp;&lt;/td&gt;' x ($confcolumns - $column),"\&lt;/tr&gt;\n";
 }
 print '&lt;/table&gt;';
 }
}
sub print_error($@)
{
 my $q = shift;
 print $q-&gt;header(),
 $q-&gt;start_html(
 -title =&gt; 'MRTG/RRD index.cgi - Script error',
 -bgcolor =&gt; '#ffffff'
 ),
 $q-&gt;h1('Script Error'),
 @_, $q-&gt;end_html();
 exit 0;
}
sub intmax(@)
{
 my (@p) = @_;
 my $max = 0;
 foreach my $n (@p) {
 $max = int($n) if defined $n and int($n) &gt; $max;
 }
 return $max;
}
sub yesorno($)
{
 my $opt = shift;
 return 0 unless defined $opt;
 return 0 if $opt =~ /^((no?)|(false)|0)$/i;
 return 1;
}
sub getdirwriteable($$)
{
 my ($base, $sub) = @_;
 $base .= $MRTG_lib::SL . $sub if $sub;
 ensureSL(\$base);
 if (!-w $base) {
 if ($^O =~ m/Win/i) {
 $base = $ENV{'TEMP'};
 $base = $ENV{'TMP'} unless $base;
 $base = $MRTG_lib::SL unless $base;
 ensureSL(\$base);
 } else {
 $base = '/tmp/';
 }
 }
 return $base;
}
use IO::File;
sub pngstring() { return chr(137)."PNG".chr(13).chr(10).chr(26).chr(10); };
sub getpngsize($)
{
 my ($file) = @_;
 my $fh = new IO::File $file;
 return () unless defined $fh;
 my $line;
 if (sysread($fh, $line, 8) != 8 or $line ne pngstring()) {
 $fh-&gt;close;
 return ();
 }
 CHUNKS: while(1) {
 last CHUNKS if (sysread($fh, $line, 8) != 8);
 my ($chunksize, $type) = unpack "Na4", $line;
 if ($type ne "IHDR") {
 last CHUNKS if (sysread($fh, $line, $chunksize + 4) != $chunksize + 4);
 next CHUNKS;
 }
 last CHUNKS if (sysread($fh, $line, 8) != 8);
 $fh-&gt;close;
 my ($x, $y) = unpack("NN", $line);
 return ('-width' =&gt; "$x", '-height' =&gt; "$y");
 }
 $fh-&gt;close;
 return ();
}
# this data contains a small png with the text:
# "error: cannot create graph"
sub errorpng()
{
 return (
 137,80,78,71,13,10,26,10,0,0,0,13,73,72,68,82,0,0,0,187,0,0,0,29,4,3,0,
 0,0,0,251,0,170,0,0,0,4,103,65,77,65,0,0,177,143,11,252,97,5,0,0,0,30,80,
 76,84,69,255,0,0,255,93,93,255,128,128,255,155,155,255,176,176,255,195,
 195,255,212,212,255,227,227,255,241,241,255,255,255,17,191,146,253,0,0,
 0,56,116,69,88,116,83,111,102,116,119,97,114,101,0,88,86,32,86,101,114,
 115,105,111,110,32,51,46,49,48,97,32,32,82,101,118,58,32,49,50,47,50,57,
 47,57,52,32,40,80,78,71,32,112,97,116,99,104,32,49,46,50,41,221,21,46,73,
 0,0,2,40,73,68,65,84,120,218,237,147,177,107,219,64,20,198,159,34,233,92,
 109,130,180,132,108,55,180,78,189,169,113,8,220,166,80,106,208,230,102,
 48,237,118,56,246,153,219,28,7,2,183,165,77,23,109,142,101,157,244,254,
 219,62,73,198,113,106,211,150,144,108,254,166,143,167,79,63,221,125,167,
 3,216,107,175,189,94,65,136,56,223,26,58,166,224,207,71,6,114,3,175,148,
 220,10,136,66,47,183,134,44,254,115,114,48,252,55,126,87,192,92,248,24,
 254,237,173,70,110,246,95,120,182,184,17,231,242,4,103,160,79,34,213,12,
 117,148,224,204,205,211,159,96,205,18,24,254,162,26,105,189,198,66,29,149,
 96,80,10,172,29,64,154,91,55,83,30,22,92,124,43,33,152,60,86,75,229,196,
 172,204,68,42,83,85,114,157,70,4,97,182,218,121,121,150,187,56,41,0,111,
 75,254,253,54,141,197,13,64,203,154,184,138,78,114,223,158,47,222,229,156,
 28,128,95,104,235,150,182,115,159,244,133,53,211,160,208,215,27,71,43,153,
 253,36,238,223,23,96,250,250,42,84,43,188,55,58,180,174,117,16,202,208,
 156,166,97,114,87,237,185,211,239,72,138,230,78,217,30,126,200,220,204,
 35,199,225,227,67,139,240,167,189,11,33,197,244,120,17,60,180,178,39,229,
 60,128,136,217,156,74,211,117,227,53,30,18,164,29,211,115,11,186,155,1,
 155,87,120,93,157,123,204,104,77,145,65,194,215,14,68,228,90,119,9,30,173,
 148,108,22,72,246,20,63,167,4,91,64,71,234,213,48,116,204,89,193,31,241,
 57,28,173,240,46,167,104,214,237,30,229,135,21,158,92,8,162,239,85,209,
 100,22,72,17,251,187,241,94,93,14,64,151,6,250,171,143,108,225,173,241,
 235,114,146,233,177,172,162,142,13,174,91,85,57,228,160,41,39,3,205,147,
 166,156,77,60,98,86,227,65,163,13,117,243,189,35,196,31,30,150,107,124,
 36,176,228,111,170,67,71,12,235,232,172,133,101,118,128,146,28,253,160,
 77,52,193,84,138,20,227,77,188,82,106,232,73,104,115,120,171,46,160,71,
 3,26,58,131,49,135,193,229,136,238,141,130,17,244,184,171,46,193,165,39,
 78,239,170,142,142,67,71,125,30,194,32,38,71,249,193,200,82,212,31,183,
 99,241,101,76,247,207,219,125,223,158,41,49,111,126,134,202,70,47,9,110,
 228,151,230,238,21,241,123,237,245,162,250,13,181,158,203,16,233,3,210,
 153,0,0,0,7,116,73,77,69,7,208,1,19,13,28,15,223,54,180,209,0,0,0,0,73,
 69,78,68,174,66,96,130
 );
}
sub gettextpic($) {
 my ($text) = @_;
 my @textsplit = split(/\n/, $text);
 my $len = 0;
 my $max = sub { $_[0] &gt; $_[1] ? $_[0] : $_[1] };
 my @rrdargs;
 foreach (@textsplit) {
 $len = &amp;$max($len, length($_));
 push @rrdargs, "COMMENT:$_\\l";
 }
 eval { require GD; 1; };
 unless ($@) {
 my $ys = @textsplit * (GD::gdMediumBoldFont()-&gt;height + 5);
 my $xs = $len * GD::gdMediumBoldFont()-&gt;width();
 my $im = new GD::Image($xs + 20, $ys + 20);
 my $back = $im-&gt;colorAllocate(255,255,255);
 $im-&gt;transparent($back);
 my $red = $im-&gt;colorAllocate(255,0,0);
 $im-&gt;filledRectangle(0,0,$xs-1,$ys-1,$back);
 my $starty = 10;
 foreach $text (@textsplit) {
 $im-&gt;string(GD::gdMediumBoldFont(), 10, $starty, $text, $red);
 $starty += 5 + GD::gdMediumBoldFont()-&gt;height;
 }
 binmode(STDOUT);
 if ($GD::VERSION lt '1.20') {
 #eval 'print $im-&gt;gif';
 return ($im-&gt;gif(), 'image/gif');
 } elsif ($GD::VERSION ge '1.20') {
 return ($im-&gt;png(), 'image/png');
 }
 }
 if ($ENV{MOD_PERL}) {
 # forking a RRDs child doesn't work with mod_perl
 return (pack("C*", errorpng()), 'image/png');
 }
 # create a graphic with rrdtool
 $len = &amp;$max($len*6-60,50);
 unshift @rrdargs, ('-', '-w', $len, '-h', 10, '-c', 'FONT#ff0000');
 my $pid = open(P, "-|");
 unless (defined $pid) {
 return (pack("C*", errorpng()), 'image/png');
 }
 unless ($pid) {
 RRDs::graph(@rrdargs);
 exit 0;
 }
 local $/ = undef;
 my $png = &lt;P&gt;;
 close P;
 unless (defined $png) {
 return (pack("C*", errorpng()), 'image/png');
 }
 return ($png, 'image/png');
}
sub log_rrdtool_call($$@) {
 my ($logfile, $error, @args) = @_;
 return unless yesorno($logfile);
 unless (open(LOG, '&gt;&gt;'.$logfile)) {
 print STDERR "cannot log rrdtool call: $!\n";
 return;
 }
 print LOG "\n# call to rrdtool:\nrrdtool ";
 foreach my $arg (@args) {
 if (ref($arg) eq 'ARRAY') {
 print LOG join(' ',@$arg),' ';
 } else {
 print LOG $arg," ";
 }
 }
 print LOG "\n";
 if ($error) {
 print LOG "# gave ERROR: $error\n";
 } else {
 print LOG "# completed without error\n";
 }
 close LOG;
}
```
Change the permission of the CGI file
```bash
[root@localhost]# chmod 777 /var/www/cgi-bin/14all.cgi
```

Note: Please make sure to allow the httpd in your SELinux &amp; IPtables or disable both SELinux &amp; IPtables
Its done...... now lets test our MRTG installation.

Open the browser window &amp;  type in http://localhost/mrtg or http://serverIP/mrtg

Thanks
Guys please share your comments about the blog  if you found this useful also let me know if you faced any issues during the process.

Enjoy.....
