
# Install Grafana on Debian or Ubuntu

    sudo apt-get install -y apt-transport-https
    sudo apt-get install -y software-properties-common wget
    sudo wget -q -O /usr/share/keyrings/grafana.key https://apt.grafana.com/gpg.key



# Stable release

    echo "deb [signed-by=/usr/share/keyrings/grafana.key] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list


# Update the list of available packages

    sudo apt-get update

# Install the latest OSS release:

    sudo apt-get install grafana

# Start the Grafana server
 
    sudo systemctl start grafana-server

# Enable the Grafana server

     sudo systemctl start grafana-server


## add inbound traffic rule for grafana-server on port no. 3000 (with selecting myIP)

### now copy server ip and paste in the browser with 3000 port no. (ip:3000) 

#### you will see the grafana ui login page 

    username: admin
    pass: admin

#### then change pass.

##### as soos as you login you will see the add data source click and you will see there is no data source because till yet we didnt installed loki and promtail

# Install docker

    sudo apt-get install docker.io -y

##### then 
   
    sudo usermod -aG docker $USER

#### then reboot the machine to apply the changes of docker.
   
    sudo reboot

#### Wait for a minute then login with your ssh client to the server


# create dir. to store the configuration file

    sudo mkdir grafana_configs

    cd grafana_configs

## paste the below command to download the configuration files for loki and promtail.

### for Loki: 
    wget https://raw.githubusercontent.com/grafana/loki/v2.8.0/cmd/loki/loki-local-config.yaml -O loki-config.yaml

### for promtail: 

    wget https://raw.githubusercontent.com/grafana/loki/v2.8.0/clients/cmd/promtail/promtail-docker-config.yaml -O promtail-config.yaml

# Now, to run Loki we have to create docker container

    docker run -d --name loki -v $(pwd):/mnt/config -p 3100:3100 grafana/loki:2.8.0 --config.file=/mnt/config/loki-config.yaml

## to see the container :

    docker ps


### now, add port no inthe inbound rule for loki container (3100) (with myIP)

##### then hit the server ip with portno.and give/ready <ip:3100/ready>

##### you will see the message on the screen just hit refresh till you see the message ready

![screenshot1](<Screenshot (308).png>)

![screenshot2](<Screenshot (309).png>)

###### and if you write metrics in front of your url then you will be able to see the metrics of Loki.

<ip:3100/metrics>

![screenshot3](<Screenshot (310).png>)

# now install promtail using docker container(also we link the Loki container with promtail container to send logs from promtail to Loki )

    docker run -d --name promtail -v $(pwd):/mnt/config -v /var/log:/var/log --link loki grafana/promtail:2.8.0 --config.file=/mnt/config/promtail-config.yaml

#### then 

    docker ps

##### go to the grafana welcome page and click add data source

![screenshot4](<Screenshot (311).png>)

## below you will see Loki > click on it

![screenshot5](<Screenshot (312).png>)

## you will see something like this:

![screenshot6](<Screenshot (313).png>)

### then go below and add 
    
    http://localhost:3100/

### [in URL under connection and below click save and test]

![screenshot7](<Screenshot (315).png>)


### Once you hit save & test button then click on Explore view

![screenshot8](<Screenshot (316).png>)


### this ui is very important withit you'll come to know which plugins or injectors and added.

![screenshot9](<Screenshot (317).png>)


### now click on select labels > select job > click on select value (you will see varlogs)

![screenshot10](<Screenshot (318).png>)

### and click Run query you will see all the logs of your system in grafana dashboard

![screenshot11](<Screenshot (319).png>)


### you can also get particular type of logs. to do this goto your server terminal and follow below steps:

![alt text](<Screenshot (320).png>)
### copy container id of promtail and hit below cmd
![alt text](<Screenshot (321).png>)
    
    cd 
    cd /var/log/grafana
    pwd

#### copy the path

    cd
    cd grafana_configs/
    vim promtail-config.yaml

#### and add these lines below

     - targets:
        - localhost
       labels:
        job: grafanalogs
        __path__: /var/log/grafana/*log

![alt text](<Screenshot (324).png>)

####  save it
    
    esc

    :wq!
    
## after saving hit below cmd to list the container:

    docker ps

![alt text](<Screenshot (321)-1.png>)

![alt text](<Screenshot (322).png>)


    docker restart <promtail container ID>


![alt text](<Screenshot (323).png>)

#### now goto grafana dashboard and refresh the page

#### now click on select value you will see the (grafanalogs) over there > select it and click run query you will see below all the grafana logs

![alt text](<Screenshot (325).png>)

![alt text](<Screenshot (326).png>)

#### now lets add these logs to dashboard

### to do that click on add to dashboard > click on open dashboard

![alt text](<Screenshot (327).png>)

### click on add > visualization

![alt text](<Screenshot (328).png>)

# now install nginx on your server and we will show the graph  of it like how many times the nginx is been used during installation

    sudo apt-get install nginx -y

### now goto same dashboard 

## below select jobs under select labels > varlogs under select value > type nginx under line container > then click operations

![alt text](<Screenshot (329).png>)

![alt text](<Screenshot (330).png>)

![alt text](<Screenshot (331).png>)

### then click apply button 

![alt text](<Screenshot (333).png>)

# hence you will see the final dashboard in this way

![alt text](<Screenshot (335).png>)