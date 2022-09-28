FROM containers.intersystems.com/intersystems/iris:2022.1.0.209.0
RUN cp /usr/irissys/bin/irisdb /usr/irissys/bin/iriscp

USER root

RUN setcap cap_ipc_lock+ep /usr/irissys/bin/iriscp

USER 51773