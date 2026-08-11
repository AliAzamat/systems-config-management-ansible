# Converging the fleet

Run the whole baseline against the inventory:

    ansible-playbook -i inventory.ini site.yml

## First run — a fresh node

    PLAY [Baseline the trading fleet] **********************************
    TASK [baseline : Standardized packages are installed] ***  changed
    TASK [baseline : A dedicated service user exists] *********  changed
    TASK [baseline : The app config is rendered from template]   changed
    TASK [baseline : The trading-gateway service is ...] ******  changed
    RUNNING HANDLER [baseline : Restart trading-gateway] ******  changed
    PLAY RECAP ********************************************************
    node01 : ok=4  changed=4  unreachable=0  failed=0

## Second run — nothing to do

    PLAY RECAP ********************************************************
    node01 : ok=4  changed=0  unreachable=0  failed=0

changed=0 on the second run is the proof: the node is at the desired state and
the playbook is safe to run again, forever. Schedule it (cron, AWX, a CI job)
and any manual drift gets corrected on the next pass.
