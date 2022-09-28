# IRIS-Docker-HugePages
How to make use of  HugePages in IRIS2022 from within a docker container

Directories
./communityEdition: demo using the IRIS Community edition 
./IRISWithLicense: demo using IRIS with a License

## QuickStart

Reserve some  HugePages in the Host OS (Temporary)
* verify that hugepages are enabled or enable if necesary
`cat /sys/kernel/mm/transparent_hugepage/enabled`
[always] madvise never
`sudo echo always > /sys/kernel/mm/transparent_hugepage/enabled`
* Verify HugePages Size (2 MB in this example)
`grep Hugepagesize /proc/meminfo`
Hugepagesize:       2048 kB

* Verify how many allocated
`cat /proc/sys/vm/nr_hugepages`
or 
`sysctl vm.nr_hugepages`

* Allocate some (350MB)
sudo sysctl -w vm.nr_hugepages=175
* community Edition demo

`cd ./communityEdition`
`docker-compose up`
You need to see this:
'09/28/22-12:07:29:340 (1149) 0 [Generic.Event] Allocated 306MB shared memory using *Huge Pages*
irislp  | 09/28/22-12:07:29:340 (1149) 0 [Generic.Event] 100MB global buffers, 44MB routine buffers, 64MB journal buffers, 9MB buffer descriptors, 72MB heap, 5MB ECP, 10MB miscellaneous
'

* Licenses Version demo
Same as community, but
- Need to login with WRC password to containers.intersystems.com to be able to login and pull IRIS 
- Copy and iris.key file in the ./license subdirectory

## How To Use HugePages from IRIS
