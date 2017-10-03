# icinga2-update
A simple command line utility to simplify updating status of passive Icinga2 service checks via the Icinga2 API. It is intended to be after cron jobs or as a post execution/failure job in systemd. It can also be used to update service status manually.

The main usecase is to get easy monitoring of cron jobs in Icinga2. This is achieved by adding service that executes a dummy check failing the service if no status is received within `check_interval`. You then update the status of these services by using `icinga2-update`.

## Python 3 package requirements
* requests
* yaml

## Icinga2 configuration
An example Service object for Icinga2:

    apply Service for (external_check => config in host.vars.external_checks) {
        import "generic-service-custom"

        /*
        Execute dummy check and return unknown if passive update have not been received
        since check_interval. Because of this, active checks should not be disabled, even
        if this service is only updated passively.
        */
        check_command = "dummy"
        check_interval = config.check_interval

        /* Set the state to CRITICAL (2) if freshness checks fail. */
        vars.dummy_state = 2

        /* Use a runtime function to retrieve the last check time and more details. */
        vars.dummy_text = {{
            var service = get_service(macro("$host.name$"), macro("$service.name$"))
            var lastCheck = DateTime(service.last_check).to_string()

            return "No check results received. Last result time: " + lastCheck
        }}
    }

Example usage in Host object:

    vars.external_checks["backup"] = {
        check_interval = 2d
    }

    vars.external_checks["backup verification"] = {
        check_interval = 60d
    }

## Usage
Create a configuration file based on the (example)[src/config.yml.example]. Either give the path to the configuration file with `--config` or place a file named `.icinga2-upate` in the users homedir (should work on both Windows and Linux)

    icinga2-update --help
    
Examples:

    # Shell
    run_some_command; icinga2-update --service <service_name> --exit-code $?
    
    # Systemd
    ExecStartPost=/usr/local/bin/icinga2-update --service <service_name> --systemd
    
    # Manual
    icinga2-update --service <service_name> --status-code 0 --status-msg "Job OK"
    
Valid status codes are determined by Icinga2 and are currently:

    0 - OK
    1 - Warning
    2 - Critical
    3 - Unknown

If `--hostname` is not provided the FQDN of the local host is used.
