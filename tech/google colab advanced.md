
#### 1. how to download files from external link in colab notebook

- colab uses an ubuntu environment underneath
-  code example using wget 
  ```bash
  !wget -O content.json {{link to file}}
```

- the saved file will not be persistent and would be deleted, so keep the command on the file to run it everytime 

