ubuntu server change network interfaces to enable access to internet:  

sudo networkctl up enp7s0 && sudo networkctl renew enp7s0  

sudo networkctl down enp7s0  

ip route  