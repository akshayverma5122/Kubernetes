

lvcreate -l 100%FREE -n lv_nfsdata nfs_vg
mkfs.ext4 /dev/nfs_vg/lv_nfsdata
systemctl daemon-reload
sudo apt install -y nfs-kernel-server
sudo chown nobody:nogroup /nfs_data
sudo chmod 777 /nfs_data

/srv/nfs/kubedata 172.16.0.0/24(rw,sync,no_subtree_check,no_root_squash)
sudo exportfs -ra
sudo systemctl enable nfs-server
sudo systemctl start nfs-server
