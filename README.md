# Create EC2 server in AWS with Ubuntu as our Ami, t2.medium, enable http and https and create server.

## Take ssh of instance and follow below steps.

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


### now, add port no. inthe inbound rule for loki container (3100) (with myIP)

##### then hit the server ip with port no. and  give /ready in front of your address. [ in this way <ip:3100/ready> ]

##### you will see the message on the screen just hit refresh till you see the message ready

![Screenshot (308)](https://github.com/sheikhnavezz/grafana_installation/assets/134357661/42af9395-bad9-43b8-a06a-5dd05360313a)


![Screenshot (309)](https://github.com/sheikhnavezz/grafana_installation/assets/134357661/4cc55699-0777-4e20-86fe-67751b459b1d)


###### and if you write metrics in front of your url then you will be able to see the metrics of Loki.

<ip:3100/metrics>

![Screenshot (310)](https://github.com/sheikhnavezz/grafana_installation/assets/134357661/571f4815-6578-45eb-80c3-6486f89dbd76)

# now install promtail using docker container(also we link the Loki container with promtail container to send logs from promtail to Loki )

    docker run -d --name promtail -v $(pwd):/mnt/config -v /var/log:/var/log --link loki grafana/promtail:2.8.0 --config.file=/mnt/config/promtail-config.yaml

#### then 

    docker ps

##### go to the grafana welcome page and click add data source

![Screenshot (311)](https://github.com/sheikhnavezz/grafana_installation/assets/134357661/03ed5fa8-117b-4007-a580-7a148aef6b9b)


## below you will see Loki > click on it

![Screenshot (312)](https://github.com/sheikhnavezz/grafana_installation/assets/134357661/389d5d67-0285-4073-b914-900bc46d69d7)


## you will see something like this:

![Screenshot (313)](https://github.com/sheikhnavezz/grafana_installation/assets/134357661/84166c19-84f2-45a6-aa29-ba50edd06537)

### then go below and add 
    
    http://localhost:3100/

### [in URL under connection and below click save and test]

![Screenshot (315)](https://github.com/sheikhnavezz/grafana_installation/assets/134357661/5c67bf99-8203-4f56-88d9-16395265f9fc)


### Once you hit save & test button then click on Explore view

![Screenshot (316)](https://github.com/sheikhnavezz/grafana_installation/assets/134357661/69105056-13e3-4f0b-93ec-5b1d75a97ac5)


### this ui is very important withit you'll come to know which plugins or injectors and added.

![Screenshot (317)](https://github.com/sheikhnavezz/grafana_installation/assets/134357661/24ecb633-ced9-4236-a39c-c78894f2d922)

### now click on select labels > select job > click on select value (you will see varlogs)

![Screenshot (318)](https://github.com/sheikhnavezz/grafana_installation/assets/134357661/e173c72a-190d-4442-ac71-19c479ae5336)


### and click Run query you will see all the logs of your system in grafana dashboard

![Screenshot (319)](https://github.com/sheikhnavezz/grafana_installation/assets/134357661/49db9589-071b-415e-b002-01180edc4a9d)

### you can also get particular type of logs. to do this goto your server terminal and follow below steps:

![Screenshot (320)](https://github.com/sheikhnavezz/grafana_installation/assets/134357661/94f8625c-88f9-4210-86ad-eee81ceb5ad7)


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

![Screenshot (324)](https://github.com/sheikhnavezz/grafana_installation/assets/134357661/b3da3a56-16e2-4f2c-99ea-91664655e8d1)

####  save it
    
    esc

    :wq!
    
## after saving hit below cmd to list the container:

    docker ps

### copy container id of promtail and hit below cmd

![Screenshot (321)](https://github.com/sheikhnavezz/grafana_installation/assets/134357661/b38de5ef-6cda-45ee-9d39-b30a53e4714f)


![alt text](<Screenshot (322).png>)


    docker restart <promtail container ID>


![Screenshot (322)](https://github.com/sheikhnavezz/grafana_installation/assets/134357661/d11113ff-e382-4b94-b7bf-36ed0152e547)

![Screenshot (323)](https://github.com/sheikhnavezz/grafana_installation/assets/134357661/bc4b237c-02dc-497f-b195-7e5ac81905e7)


#### now goto grafana dashboard and refresh the page

#### now click on select value you will see the (grafanalogs) over there > select it and click run query you will see below all the grafana logs

![Screenshot (325)](https://github.com/sheikhnavezz/grafana_installation/assets/134357661/b2b353d7-a670-4190-82ae-227242c9e67e)

![Screenshot (326)](https://github.com/sheikhnavezz/grafana_installation/assets/134357661/6a8d7161-4325-4abf-8afe-d9a6255c50cf)


#### now lets add these logs to dashboard

### to do that click on add to dashboard > click on open dashboard

![Screenshot (327)](https://github.com/sheikhnavezz/grafana_installation/assets/134357661/39e8f4c7-3c9e-45a0-be0e-82476f6c25e5)


### click on add > visualization

![Screenshot (328)](https://github.com/sheikhnavezz/grafana_installation/assets/134357661/11ceebe7-50ef-4e6e-badd-f9908ed85102)


# now install nginx on your server and we will show the graph  of it like how many times the nginx is been used during installation

    sudo apt-get install nginx -y

### now goto same dashboard 

## below select jobs under select labels > varlogs under select value > type nginx under line container > then click operations

![Screenshot (329)](https://github.com/sheikhnavezz/grafana_installation/assets/134357661/f5421995-3cd3-48fc-9b38-5947b07de081)


![Screenshot (330)](https://github.com/sheikhnavezz/grafana_installation/assets/134357661/205e0e44-2888-480b-8b5b-90a78a58d23f)


![Screenshot (331)](https://github.com/sheikhnavezz/grafana_installation/assets/134357661/b2b83302-0e65-4d67-9419-d0bffa5c904f)

### then click apply button 

![Screenshot (333)](https://github.com/sheikhnavezz/grafana_installation/assets/134357661/c5f5416b-0d04-489d-b902-f43de074181c)

# hence you will see the final dashboard in this way

![Screenshot (335)](https://github.com/sheikhnavezz/grafana_installation/assets/134357661/fb7c40cf-bc72-4373-8188-5af2336d5a81)


## Therefore, We have successfully configured our server, Integrate grafana; using Loki, and promtail on the server and visualize the logs in the form of metrics and graphs.
