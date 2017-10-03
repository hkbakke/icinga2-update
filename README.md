# icinga2-update
A simple command line utility to simplify updating status of passive Icinga2 service checks via the Icinga2 API. It is intended to be after cron jobs or as a post execution/failure job in systemd. It can also be used to update service status manually.

The main usecase is to get easy monitoring of cron jobs in Icinga2. This is achieved by adding service that executes a dummy check failing the service if no status is received within `check_interval`. You then update the status of these services by using `icinga2-update`.

## Requirements

    apt install python3-requests python3-yaml

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

    icinga2-update --help
