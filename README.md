# MinIO Easy Deploy & Mount with Docker Compose

Deploy MinIO effortlessly on one Linux host and mount it on another Linux host. This project simplifies the setup of MinIO as an object storage solution, enabling quick integration with your local file system. Perfect for developers needing efficient storage management across multiple Linux environments with minimal setup.

## Prerequisites

- Docker and Docker Compose installed on both hosts.
- Ensure `fuse` is enabled on the host where MinIO buckets will be mounted:
  ```bash
  sudo modprobe fuse
  ```

## MinIO Deployment

Deploy MinIO with the following `docker-compose.yml` to store data in `/home/minio-data`:

```yaml
version: '3'
services:
  minio:
    image: minio/minio:latest
    container_name: minio
    ports:
      - 60004:9000 # MinIO data API port
      - 60005:9001 # MinIO web console
    environment:
      TZ: Asia/Shanghai
      MINIO_ROOT_USER: <YOUR_MINIO_ROOT_USERNAME, e.g. 'admin'>
      MINIO_ROOT_PASSWORD: <YOUR_MINIO_ROOT_PASSWORD, e.g. 'password'>
    volumes:
      - <YOUR_DATA_STORAGE_DIRECTORY, e.g. '/home/minio-data'>:/data
    command: server /data --console-address ":9001"
    restart: always
```

## Mount MinIO Bucket

Mount the MinIO `public` bucket on another Linux host using the following `docker-compose.yml`:

```yaml
version: "3.9"
services:
  rclone:
    image: rclone/rclone
    container_name: oss_public
    volumes:
      - <YOUR_DATA_STORAGE_DIRECTORY, e.g. '/home/oss/public'>:/bucket_data:shared
    entrypoint: >
      /bin/sh -c "
      mkdir -p /config/rclone && 
      echo '[minio]' > /config/rclone/rclone.conf && 
      echo 'type = s3' >> /config/rclone/rclone.conf && 
      echo 'provider = Minio' >> /config/rclone/rclone.conf && 
      echo 'env_auth = false' >> /config/rclone/rclone.conf && 
      echo 'access_key_id = <YOUR_ACCESS_KEY_ID>' >> /config/rclone/rclone.conf && 
      echo 'secret_access_key = <YOUR_SECRET_ACCESS_KEY>' >> /config/rclone/rclone.conf && 
      echo 'endpoint = http(s)://<YOUR_MINIO_SERVER_IP>:<YOUR_MINIO_PORT>/' >> /config/rclone/rclone.conf && 
      rclone mount minio:/public /bucket_data --allow-non-empty
      "
    restart: always
    privileged: true
```

### Steps to Mount

1. Save the above content into a file named `docker-compose.yml`.
2. Run the following command in the directory where your `docker-compose.yml` is located:
   ```bash
   docker-compose up -d
   ```

## Important Notes

- Ensure `fuse` is loaded on the system where you are mounting the bucket.
- Replace placeholders with your actual MinIO credentials and endpoint information.
- Adjust paths as necessary to fit your environment.

Feel free to reach out if you have any questions or need further assistance!
