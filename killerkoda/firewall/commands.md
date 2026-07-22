```bash
timeout 3 nc -vz node01 80
timeout 3 nc -vz node01 22
timeout 3 nc -vz controlplane 3306
ssh node01 'timeout 3 nc -vz controlplane 22'
```