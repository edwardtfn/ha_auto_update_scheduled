#
# Verlaufsdiagramm ha autoupdates
#

blueprint info

    * domain: automation
    * input
        * backup_bool
        * restart_bool
        * schedule_entity
        * schedule_monthday
        * core_os_update_mode
        * update_exclusions
        * actions_pre_update
        * actions_pos_update
        * pause_entities
        
        * backup_timeout
        * backup_location
        * update_inclusion_mode
        * update_inclusion_entity_list


automation info

    * trigger_variables
        * input_schedule_monthday
        * input_core_os_update_mode
        * input_pause_entities
        * temp_input_update_exclusions
        * input_update_exclusions
        * update_list
        
        * input_update_inclusion_mode
        * input_update_inclusion_entity_list
    
    * trigger
        * schedule
        * HA start  (after HA restart)
        * update available
        * new day has begun

    * conditions
        * value_template: do we have any updates?
        * value_template: are we later than the blocked days at beginning from month
        * state: is the schedule state on?
        * OR
            * state: input_pause_entities_selected is not defined/true
            * state: the entity from the blueprint input "pause_entities" should also be in off state

    * variables
        * input_backup_bool                 get input
        * input_backup_timeout              get input
        * input_restart_bool                get input
        * input_schedule_monthday           get input
        * input_core_os_update_mode         get input
        * input_pause_entities              get input

        * core_update_entity                search for Core Update entites via integration
        * os_update_entity                  search for OS Update entites via integration
        * supervisor_update_entity          search for Supervisor Update entites via integration

        * temp_input_update_exclusions      get input
        * input_update_exclusions           set exclusion list according to rules from input_core_os_update_mode
                                            Rules:
                                                - All       dont add core/os entity ids to exclusion list
                                                - Ignore    add entity ids for core + os to exclusion list
                                                - patches   check the changes of core + os updates. 
        * update_list                       get all update entities with state on, exclude all exclusions

    * actions
        
        ##############################################################
        # pre-actions
        #
        
        * IF list of updates is under '1'
            message: Nothing to update
        * log
            show input variables
        * log
            "A new update is available for Home Assistant
        * log
            List of updates
        * log
            Running pre-actions
        * choose
            include sequence from input actions_pre_update
        
        ##############################################################
        # trigger backup
        #
        
        * IF input_backup_bool
            * log
                "Backing up"
            * backup_full
            * log
                "Backup triggered"
            * wait 
                for input_backup_timeout
        
        ##############################################################
        # do updates - style "normal" (exclude core, os, exclusions)
        #
        
        * recalculate all variables
        * repeat
            * condition
                - state: schedule_entity on
                - template: get all update entities, filter on state 'on', filter out core + os + exclusion entities, count, true if list is longer than zero
            * sequence
                * log
                    "Starting sequence of standard updates..."
                * variables
                    set "pending_update_list"
                * log
                    "Updating <first item of pending_update_list> of <pending_update_list>
                * update.install
                    first of pending_update_list
                * wait_template
                    wait on state of <first of pending_update_list> to be off
                    wait for max 3600
        
        ##############################################################
        # do updates - style "with core" (exclude os, exclusions)
        #
        
        * recalculate all variables
        * repeat
            * condition
                - state: schedule_entity on
                - template: get all update entities, filter on state 'on', filter out os + exclusion entities, count, true if list is longer than zero
            * sequence
                * log
                    "Starting sequence of standard updates..."
                * variables
                    set "pending_update_list"
                * log
                    "Updating <first item of pending_update_list> of <pending_update_list>
                * update.install
                    first of pending_update_list
                * wait_template
                    wait on state of <first of pending_update_list> to be off
                    wait for max 3600
        
        ##############################################################
        # do updates - style "with os" (exclude core, exclusions)
        #
        
        * recalculate all variables
        * repeat
            * condition
                - state: schedule_entity on
                - template: get all update entities, filter on state 'on', filter out os + exclusion entities, count, true if list is longer than zero
            * sequence
                * log
                    "Starting sequence of standard updates..."
                * variables
                    set "pending_update_list"
                * log
                    "Updating <first item of pending_update_list> of <pending_update_list>
                * update.install
                    first of pending_update_list
                * wait_template
                    wait on state of <first of pending_update_list> to be off
                    wait for max 3600
        
        ##############################################################
        # do updates - rest of 'em
        #
        
        * IF
            * condition:
                - state: schedule_entity on
            * then:
                * log
                    "Updating all remaining items"
                * update.install
                    all update entities, filter on state:on, filter on updates_list, filter out exclusions
                * wait_template
                    wait on state of all update entities which were on to be off
                    wait for max 3600
        
        ##############################################################
        # finishing
        #
        
        * log
            "Finishing update process"
        * log
            List of remaining updates
        
        ##############################################################
        # restart home asisstant
        #
        
        * IF
            * condition
                - input_restart_bool
                - get update entities, 
                  get which have a release_summary,
                  search in release_summary for ha alert type error
                  get the entity id
                  make it a list
                  count
                  check if length of list is longer than zero
            * then
                * log
                    "Restarting HA"
                * hassio.host_reboot
        
        ##############################################################
        # post-update actions
        #

        * log
            "running post-update actions"
        * include post_update actions

        ##############################################################
        # Finished!
        #

        * log
            "We are finished!"