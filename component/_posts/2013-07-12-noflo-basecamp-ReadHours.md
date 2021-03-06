---
  title: "ReadHours"
  library: "noflo-basecamp"
  layout: "component"

---

```coffeescript
noflo = require "noflo"
base = require "../lib/BasecampComponent.coffee"

class ReadHours extends base.BasecampComponent
  constructor: ->
    @start = false
    do @basePortSetup

    @inPorts.start = new noflo.Port()

    @outPorts =
      out: new noflo.Port()

    @inPorts.start.on "data", =>
      @start = true
      do @readHours if @hostname and @apikey
    @inPorts.apikey.on "disconnect", =>
      do @readHours if @hostname and @start
    @inPorts.hostname.on "disconnect", =>
      do @readHours if @apikey and @start

  readHours: ->
    @get "/time_entries/report.xml", (data) =>
      @parseHours data

  parseHours: (data) ->
    target = @outPorts.out
    id = "https://#{@hostname}/"
    @parse data, (parsed) ->
      target.beginGroup id
      target.send entry for entry in parsed['time-entries']['time-entry']
      target.endGroup()
      target.disconnect()
    
exports.getComponent = -> new ReadHours

```