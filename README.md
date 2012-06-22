SMF - Service Management Facility
=================================

Expose smf(5) to Node.js for Solaris/Illumos based operating systems

Install
------

Install locally to use as a module

    npm install smf

Install globally to use the command-line tool `smf`

    npm install -g smf

Usage
-----

as a module

``` js
var smf = require('smf');
```

as a command-line tool

    ~$ smf [ service-name ]

Functions
---------

### smf.svcs(callback(err, services))

Call `smf.svcs` with a single callback argument to get a list of service fmri's
on the system.

### smf.svcs('name', callback(err, svc))

Call `smf.svcs` with a service name and a callback to get information
about the given service.

---

### smf.svcadm('action', 'name')

Do a given action to the given name.  For example `smf.svcadm('disable', 'apache');`.
This command will return immediately, without waiting for the service to be disabled
fully, and without checking error codes.

### smf.svcadm('action', 'name', {option:value}, callback(err, code))

Do a given action to the given name with additional options.  When the
command is finished the callback will be called with an error message
(or null) as the first argument, and the exit code of the svcadm(1M)
call.

_Options_

1.  `{temporary:true}` - Temporarily disable or enable a service
2.  `{wait:true}` - Wait for the service to change state before executing `callback`

Options and callbacks are optional. See EXAMPLES for more information.

Example
-------

### smf.svcs()

``` js
var svcs = require('smf').svcs;
svcs(function(err, services) {
  // services => list of all service fmri's on the system
  console.log('%d services found.', services.length);
  console.log('Looking up last service found...');
  svcs(services[services.length - 1], function(err, svc) {
    // svc => associative array of service information
    if (err) throw err;
    console.log(svc);
  });
});
```

    161 services found.
    Looking up last service found...
    { fmri: 'svc:/system/boot-archive:default',
        name: 'check boot archive content',
        enabled: 'true',
        state: 'online',
        next_state: 'none',
        state_time: 'Wed Apr 25 01:32:33 2012',
        logfile: '/var/svc/log/system-boot-archive:default.log',
        restarter: 'svc:/system/svc/restarter:default',
        dependency: [ 'require_all/none svc:/system/filesystem/root (online)' ] }

### smf.svcadm()

``` js
var svcadm = require('smf').svcadm;

// disable nginx without wating or confirmation
svcadm('disable', 'nginx');

// enable nginx and callback with results
svcadm('enable', 'nginx', function(err, code) {
  if (err) throw err;
  console.log('Return code %d', code);
});

// disable nginx, wait for it to enter 'disabled' state, and callback
svcadm('disable', 'nginx', {wait:true}, function(err, code) {
  if (err) throw err;
  console.log('Return code %d', code);
});

// restart nginx without waiting or confirmation
svcadm('restart', 'nginx');

// temporarily disable nginx without waiting or confirmation
svcadm('disable', 'nginx', {temporary:true});
```

Command Line
------------

If you install this module globally you will also get the `smf`
tool to expose information from svcs(1).  Run it without
any arguments to get a newline separated list of service fmri's
on the system.  Run it with a service-name argument to get a json
object of information about the given service.

Example
-------

    root@dave.voxer.com# smf | tail -3
    svc:/milestone/devices:default
    svc:/system/device/local:default
    svc:/system/boot-archive:default


    root@dave.voxer.com# smf name-service-cache | json

``` json
{
  "fmri": "svc:/system/name-service-cache:default",
  "name": "name service cache",
  "enabled": "true",
  "state": "online",
  "next_state": "none",
  "state_time": "June 19, 2012 10:06:13 PM UTC",
  "logfile": "/var/svc/log/system-name-service-cache:default.log",
  "restarter": "svc:/system/svc/restarter:default",
  "contract_id": "5714820",
  "dependency": [
    "require_all/restart file://localhost/etc/nscd.conf (online) file://localhost/etc/nsswitch.conf (online)",
    "require_all/none svc:/system/filesystem/minimal (online)",
    "optional_all/refresh svc:/network/location:default (disabled)",
    "optional_all/none svc:/system/rbac (online)"
  ],
  "process": [
    {
      "pid": "3699",
      "cmd": "/usr/sbin/nscd"
    }
  ],
  "config": "(application)",
  "config/enable_per_user_lookup": "(boolean) = true",
  "config/per_user_nscd_time_to_live": "(integer) = 120"
}
``` json

License
-------

MIT Licensed
