# Logstash filtering


## Description
Custom filters for [Logstash](https://www.elastic.co/logstash/) to parse JSON, XML and Kafka logs.

# Table of contents
1. [Requierments](#requirements)
2. [Usage](#usage)
3. [Filters](#filters)
    1. [JSON](#json)
    2. [XML](#xml)
    3. [KAFKA](#kafka)


## Requirements <a name="requirements"></a>
To use these filters, you are required to have configured the next services from [ELK](https://www.elastic.co/what-is/elk-stack):
* [Elastic](https://www.elastic.co/)
  * [Docs](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)
  * [Download](https://www.elastic.co/start)
* [Kibana](https://www.elastic.co/kibana/)
  * [Docs](https://www.elastic.co/guide/en/kibana/current/index.html)
  * [Download](https://www.elastic.co/start)
* [Logstash](https://www.elastic.co/logstash/)
  * [Docs](https://www.elastic.co/guide/en/logstash/current/index.html)
  * [Download](https://www.elastic.co/downloads/logstash)



## Usage <a name="usage"></a>
First of all, it is __important__ to know where to save the configuration file for Logstash. The configuration file is usually saved in the root of the file and Logstash will start with the following command: 

```powershell
bin\logstash -f config_file_name.conf
```
Please check the [documentation](https://www.elastic.co/guide/en/logstash/current/running-logstash-command-line.html) to learn more.

If you are running the ELK Stack using Docker, be aware that you need to share the configuration file with your container. This is a simple approach:
```compose
logstash:
    image: logstash:7.14.0
    ports:
      - '5000:5000'
    volumes:
      - type: bind
        source: ./logstash_pipeline/
        target: /usr/share/logstash/pipeline
        read_only: true
```
In the folder where you have the docker-compose.yml file, you will create a new folder *logstash_pipeline* (this is just an example) and a configuration file after the structure:
```
root
| docker-compose.yml
|
└── logstash_pipeline
    | ports.conf
```
After you have configured your environment, you will use the [filters](#filters) from below to parse the documents that are coming through your inputs configured in Logstash configuration file.

Your file might have the next structure as in example below: 
```Properties
input{
    tcp{
        port => 5000
    }
}


filter {}

output{
    elasticsearch{
        hosts => ["<elasticsearch_host>:<elasticsearch_port>"] #Example: localhost:9200 OR the IP for ElasticSearch
        index => "<index_name>"
    }
}
```
After the configuration file is properly filled with the missing information, you need to restart __Logstash__ service to apply the new filter.

## Filters <a name="filters"></a>
In many cases it is used ```remove_field => ["message"]``` to delete the original message considering that it is now segmented by filters in more fields.

To filter your logs, please use the right filter from below.

Please check the [documentation](https://www.elastic.co/guide/en/logstash/current/filter-plugins.html) to learn more.


#### JSON <a name="json"></a>
```Properties
filter {
    json {
		source => "message"
	}
    mutate {            
               remove_field => ["message"]
       }
}
```

#### XML <a name="xml"></a>
```Properties
filter {
    grok{
        match => {
            "message" => ["(?<xml><\?xml[\s\S]*?<\SAuditMessage>)"]
        }
    }
    grok{
        match => {
            "message" => ["(?<ip_region>ip\-(([1-9]?\d|[12]\d\d)\-){3}([1-9]?\d|[12]\d\d)[\S]*)"]
        }
    }
    grok {
		match => { "message" => "%{TIMESTAMP_ISO8601:logdate}"}
    }
    xml {
      source => ["xml"]
      target => "audit_log"
    }
    mutate {            
            remove_field => ["message"]
            remove_field => ["xml"]
       }
}
```

#### KAFKA <a name="kafka"></a>
```Properties
filter {
    grok{
        match => {
            "message" => "%{TIMESTAMP_ISO8601:logdate} %{LOGLEVEL:loglevel} %{GREEDYDATA:log_message} "
        }
    }
     mutate {            
            remove_field => ["message"]
       }
}
```
