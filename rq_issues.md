## netbox-rq  service not starting

Fix 

```
sudo -i
cd /opt/netbox
source venv/bin/activate
pip3 uninstall rq
pip3 install  "rq==1.13.0"
systemctl restart netbox netbox-rq
```
### Web UI issues
```
./manage.py collectstatic
```
