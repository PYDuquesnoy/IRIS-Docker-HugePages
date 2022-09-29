FROM containers.intersystems.com/intersystems/iris-community:2022.1.0.209.0
#Add all your optional container add-ons here
#Install software in IRIS here


#Restart iris if needed
#USER 51773
#RUN iris start IRIS && iris stop IRIS quietly


#---Finally, execute this code 
RUN cp /usr/irissys/bin/irisdb /usr/irissys/bin/iriscp
USER root
RUN setcap cap_ipc_lock+ep /usr/irissys/bin/iriscp
USER 51773