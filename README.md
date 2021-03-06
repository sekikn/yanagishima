# yanagishima

yanagishima is a Web UI for presto/hive.

![preview](v4.gif)

# Features
* easy to install
* easy to use
* run query(Ctrl+Enter)
* query history
* query bookmark
* show query execution list
* kill running query
* show columns
* show partitions
* show query result data size
* show query result line number
* TSV download
* CSV download
* show presto view ddl
* share query
* share query result
* search table
* handle multiple presto/hive clusters
* auto detection of partition key
* show progress of running query
* query parameters substitution
* insert chart
* format query(Ctrl+Shift+Enter)
* convert from TSV to values query(presto only)
* function, table completion(Ctrl+Space, presto only)
* validation(Shift+Enter, presto only)
* export/import history
* export/import bookmark
* desktop notification

# Limitation

* paging results is not supported
* Access Control
* Authentication

# Versions
* 7.0
  * support hive on MapReduce(yanagishima executes ```set mapreduce.job.name=...```)
  * metadata of 7.0 is NOT compatible with 6.0, so migration is required
  * migration process is as follows
  ```
  cp data/yanagishima.db data/yanagishima.db.bak
  sqlite3 data/yanagishima.db
  sqlite> create table query_new (datasource text, engine text, query_id text, fetch_result_time_string text, query_string text, primary key(datasource, engine, query_id));
  sqlite> insert into query_new select datasource, 'presto', query_id, fetch_result_time_string, query_string from query;
  sqlite> alter table query rename to query_old;
  sqlite> alter table query_new rename to query;
  sqlite> create table publish_new (publish_id text, datasource text, engine text, query_id text, primary key(publish_id));
  sqlite> insert into publish_new select publish_id, datasource, 'presto', query_id from publish;
  sqlite> alter table publish rename to publish_old;
  sqlite> alter table publish_new rename to publish;
  sqlite> create table bookmark_new (bookmark_id integer primary key autoincrement, datasource text, engine text, query text, title text);
  sqlite> insert into bookmark_new select bookmark_id, datasource, 'presto', query, title from bookmark;
  sqlite> alter table bookmark rename to bookmark_old;
  sqlite> alter table bookmark_new rename to bookmark;
  If you confirmed, drop table bookmark_old;
  ```
* 6.0
  * support bookmark title, so add title column to bookmark table
  * metadata of 6.0 is NOT compatible with 5.0, so migration is required
  * migration process is as follows
  ```
  cp data/yanagishima.db data/yanagishima.db.bak
  sqlite3 data/yanagishima.db
  sqlite> create table if not exists bookmark_new (bookmark_id integer primary key autoincrement, datasource text, query text, title text);
  sqlite> insert into bookmark_new select bookmark_id, datasource, query, null from bookmark;
  sqlite> alter table bookmark rename to bookmark_old;
  sqlite> alter table bookmark_new rename to bookmark;
  If you confirmed, drop table bookmark_old;
  ```

# Quick Start
```
wget https://bintray.com/artifact/download/wyukawa/generic/yanagishima-7.0.zip
unzip yanagishima-7.0.zip
cd yanagishima-7.0
vim conf/yanagishima.properties
nohup bin/yanagishima-start.sh >y.log 2>&1 &
```
see http://localhost:8080/

# Configuration

You need to edit conf/yanagishima.properties.
```
# yanagishima web port
jetty.port=8080
# 30 minutes. If presto query exceeds this time, yanagishima cancel the query.
presto.query.max-run-time-seconds=1800
# 1GB. If presto query result file size exceeds this value, yanagishima cancel the query.
presto.max-result-file-byte-size=1073741824
# you can specify freely. But you need to specify same name to presto.coordinator.server.[...] and presto.redirect.server.[...] and catalog.[...] and schema.[...]
presto.datasources=your-presto
# presto coordinator url
presto.coordinator.server.your-presto=http://presto.coordinator:8080
# almost same as presto coordinator url. If you use reverse proxy, specify it
presto.redirect.server.your-presto=http://presto.coordinator:8080
# presto catalog name
catalog.your-presto=hive
# presto schema name
schema.your-presto=default
# if query result exceeds this limit, to show rest of result is skipped
select.limit=500
# http header name for audit log
audit.http.header.name=some.auth.header
# limit to convert from tsv to values query
to.values.query.limit=500
# authorization feature
check.datasource=false
hive.jdbc.url.your-hive=jdbc:hive2://localhost:10000/default;auth=noSasl
hive.jdbc.user.your-hive=yanagishima-hive
hive.jdbc.password.your-hive=yanagishima-hive
hive.query.max-run-time-seconds=3600
hive.query.max-run-time-seconds.your-hive=3600
resource.manager.url.your-hive=http://localhost:8088
sql.query.engines=presto,hive
hive.datasources=your-hive
hive.disallowed.keywords.your-hive=insert,drop
# 1GB. If hive query result file size exceeds this value, yanagishima cancel the query.
hive.max-result-file-byte-size=1073741824
```

If you use a single presto cluster, you need to specify as follows.
```
jetty.port=8080
presto.query.max-run-time-seconds=1800
presto.max-result-file-byte-size=1073741824
select.limit=500
to.values.query.limit=500
presto.datasources=your-presto
presto.coordinator.server.your-presto=http://presto.coordinator:8080
presto.redirect.server.your-presto=http://presto.coordinator:8080
catalog.your-presto=hive
schema.your-presto=default
sql.query.engines=presto
```

If you want to handle multiple presto clusters, you need to specify as follows.
```
presto.datasources=presto1,presto2
presto.coordinator.server.presto1=http://presto1.coordinator:8080
presto.redirect.server.presto1=http://presto1.coordinator:8080
presto.coordinator.server.presto2=http://presto2.coordinator:8080
presto.redirect.server.presto2=http://presto2.coordinator:8080
catalog.presto1=hive
schema.presto1=default
catalog.presto2=hive
schema.presto2=default
```

If you use a single hive cluster, you need to specify as follows.
```
jetty.port=8080
select.limit=500
hive.query.max-run-time-seconds=3600
hive.max-result-file-byte-size=1073741824
hive.jdbc.url.your-hive=jdbc:hive2://localhost:10000/default;auth=noSasl
hive.jdbc.user.your-hive=yanagishima-hive
hive.jdbc.password.your-hive=yanagishima-hive
resource.manager.url.your-hive=http://localhost:8088
sql.query.engines=hive
hive.datasources=your-hive
```

If you use presto and hive, you need to specify as follows.
```
jetty.port=8080
presto.query.max-run-time-seconds=1800
presto.max-result-file-byte-size=1073741824
select.limit=500
to.values.query.limit=500
presto.datasources=your-cluster
presto.coordinator.server.your-cluster=http://presto.coordinator:8080
presto.redirect.server.your-cluster=http://presto.coordinator:8080
catalog.your-cluster=hive
schema.your-cluster=default
hive.query.max-run-time-seconds=3600
hive.max-result-file-byte-size=1073741824
hive.jdbc.url.your-cluster=jdbc:hive2://localhost:10000/default;auth=noSasl
hive.jdbc.user.your-cluster=yanagishima-hive
hive.jdbc.password.your-cluster=yanagishima-hive
resource.manager.url.your-cluster=http://localhost:8088
sql.query.engines=presto,hive
hive.datasources=your-cluster
```

# Audit Logging
yanagishima doesn't have authentication feature.
but, if you use reverse proxy server like Nginx for authentication, you can add audit logging.
In this case, please specify ```audit.http.header.name``` which is http header name to be passed through Nginx.

# Start
```
bin/yanagishima-start.sh
```

# Stop
```
bin/yanagishima-shutdown.sh
```

# Requirements

* Java 8

## Build yanagishima

```
./gradlew distZip
```

## For Front-end Engineer

### File organization

|File|Description|Copy to docroot|Build index.js|
|:--|:--|:-:|:-:|
|build/index.html|SPA body|Yes||
|build/index.js|Static assets (JS/CSS/IMG)|Yes||
|build/favicon.ico|Favorite icon|Yes||
|build/share/index.html|Published result page (readonly)|Yes||
|build/error/index.html|Error page template|Yes||
|source/config.js|Config for yanagishima||Yes|
|source/core.js|SPA core||Yes|
|source/plugin.js|Vue plugin for Ace Editor||Yes|
|source/yanagishima.svg|Logo/Background image||Yes|
|source/scss/bootstrap.scss|CSS based on Bootstrap||Yes|
|webpack.config.json|Config for webpack|-|-|
|browsersync.config.json|Config for Browsersync|-|-|

### Framework/Plugin

- CSS
	- Bootstrap 4.0.0 alpha.6
	- FontAwesome 4.7.0
	- Google Fonts "[Droid+Sans](https://fonts.google.com/specimen/Droid+Sans)"
- JavaScript
	- Vue 2.3.0
	- Ace Editor 1.2.6
	- Sugar 2.0.4
	- jQuery 3.2.1
- Build/Serving tool
	- webpack 2.2.1
	- browser-sync 2.18.8

### Deep customization

#### Installation

	$ cd web
	$ npm install

#### Build/Serving and Livereload

	$ npm start
