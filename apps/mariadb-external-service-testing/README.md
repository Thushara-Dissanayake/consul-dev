# ******************** MariaDB connection testing with Service Mesh (Without HCP) ****************************

export CONSUL_HTTP_TOKEN=$(kubectl get --namespace consul secrets/consul-bootstrap-acl-token --template={{.data.token}} | base64 -d)
CONSUL_HTTP_ADDR=http://localhost:8500
export DB_URL=db-1.c5mqbengbhqr.eu-central-1.rds.amazonaws.com

k port-forward -n consul svc/consul-server 8500:8500

1. Delete ServiceDefaults of mariaDB if you have any

2. Register mariaDB as an external service

    curl -k --request PUT --data @external.json ${CONSUL_HTTP_ADDR}/v1/catalog/register --header "Authorization: Bearer 08f4d946-63b8-b0f9-44fa-a9f128652c7e"

    To degister an external service:

        curl \
            --request PUT \
            --data @external-deregister.json \
            --header "Authorization: Bearer 1452d67f-25b2-bd57-5dcc-9aba533b07de" \
            https://hcp-lzprd-fra-dev-001.private.consul.9da0a92d-8d26-4944-8a6e-bf448298575e.aws.hashicorp.cloud/v1/catalog/deregister 


3. Verify service registration
    consul catalog services

    curl ${CONSUL_HTTP_ADDR}/v1/catalog/nodes | jq
    curl ${CONSUL_HTTP_ADDR}/v1/catalog/services | jq



4. Test DB connection

    mysql --host=db-1.c5mqbengbhqr.eu-central-1.rds.amazonaws.com --user=admin --password=admin123 --database=mysql --execute='SHOW tables;'
    
    mysql -h db-1.c5mqbengbhqr.eu-central-1.rds.amazonaws.com -u admin -p -e 'SHOW databases;'
    
    If connecion is successful:

            Enter password:
        +--------------------+
        | Database           |
        +--------------------+
        | information_schema |
        | innodb             |
        | mysql              |
        | performance_schema |
        | sys                |
        +--------------------+

    If connection is unsuccessful:

    "ERROR 2013 (HY000): Lost connection to MySQL server at 'reading initial communication packet', system error: 0"