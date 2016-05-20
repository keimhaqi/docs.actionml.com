{{#template name='pio_cli_cheatsheet'}}
# PredictionIO CLI Cheatsheet

PredictionIO can be seen as 2 types of servers, one takes in and stores events&mdash;the EvnetServer&mdash;and the other serves prediction&mdash;the PredictionServer. The general non-template specific commands can be run from anywhere, in any directory but the template specific commands must be run in the directory of the specific engine-instance being used, this is because some commands rely on files (like engine.json) to be available.

#General Commands
At any point you can run `pio help some-command` to get a help screen printed with all supported options for a command.

##Start/stop

 - `pio-start-all` this can only be used reliably on a single server setup with all services on a single machine.
 - `pio-stop-all` likewise this is only for a single machine setup.
 - `pio eventserver` this starts an EventServer on port 7070 of localhost
 - `nohup pio eventserver &` this creates an EventServer as a daemon, other daemon creation commands work too, like `screen`.
 
##Status and Information

 - `pio status` this checks the config of PredictionIO and connects to the databased used, it does not connect to Spark or check the status of things like HDFS.
 - `pio app list` list information about apps the systems knows about, this is used primarily to see which collections of data are registered with the EventServer.
 - `pio app new some-appname` this creates an empty collection and a key that can be used to send events to the EventServer.
 - `pio app delete some-appname` remove app and all data from the EventServer
 - `pio app data-delete some-appname`

## Import Data

The EventServer can hold data as soon as you have created an app as above. Then you can choose to import JSON events from files (the fastest method) or use the REST API to import.

 - `pio import` will import files with JSON events to an app created with `pio app new appname`

If you want to use the REST method you will use and SDK or make raw REST post calls. To add events from a shell script you would use `curl` to post to the EventServer on port 7070 of your host. Like this:

    $ curl -i -X POST http://localhost:7070/events.json?accessKey=some-key \
    -H "Content-Type: application/json" \
    -d '{
      "event": "my_event",
      "entityType": "user"
      "entityId": "user-id",
      "targetType": "item",
      "targetEntityId": "item-id"
      "eventTime" : "2004-12-13T21:39:45.618-07:00"
    }'

Events are defined by the template so check the specific template docs for encoding data in events.

#Workflow Commands

For some pio commands you must `cd` to an engine-instance directory. This is because the `engine.json` and/or `manifest.json` are either needed or are modified. These commands implement the workflow for creating a "model" from events and launching the PredictionServer to serve queries.

##Standard Workflow

These commands must be run in this order, but can be repeated once previous commands are run. So many trains are expected after a build and many deploys of the same model are allowed.


 - `pio build` this registers the `engine.json` params with the meta-store as defined in the `pio-env.sh`, it also uses sbt to compile and create jars from the engine code. Any change to `engine.json` will only take effect after `pio build` even if the code has not changed.
 - `pio train` pulls data from the event store and creates a model
 - `pio deploy` creates a PredictionServer instance to serve query results based on the last trained model
 - `nohup pio deploy &` creates a daemon of the PredictionServer for the current engine-instance
 
##<a id='ur-workflow'></a> Universal Recommender Example

{{> urworkflow}}
 
{{/template}}