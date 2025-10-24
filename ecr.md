# Lấy code và chuyển branch cần deploy
cd ~/sabeco-app
git fetch --all
# git checkout <branch_name>
git checkout ai-analyzing-brand   
git pull

# Đảm bảo thư mục chứa:
# - Dockerfile
# - nginx.conf
# - package.json
# - yarn.lock
# - src/
# - public/

# Mục đích:
# - Đồng bộ toàn bộ branch mới nhất từ GitHub (git fetch --all).
# - Tạo và chuyển sang branch ai-analyzing-brand để làm việc.
# - Cập nhật code mới nhất trên branch đó (git pull).

# Đăng nhập AWS SSO để có quyền ECR

aws configure sso  
SSO session name (Recommended): devops
SSO start URL [None]: https://d-96675de7bc.awsapps.com
SSO region [None]: ap-southeast-1
SSO registration scopes [sso:account:access]: sso:account:access

# Kiểm tra xác thực SSO:

aws sts get-caller-identity --profile AWSDevOpsEngineer-905418169471
# Kết quả:
# {
#   "UserId": "AROA5FTZA3R7ZDQ2E3TZD:huy.dinh@emesoft.net",
#   "Account": "905418169471",
#   "Arn": "arn:aws:sts::905418169471:assumed-role/AWSReservedSSO_AWSDevOpsEngineer_58147a630e5deff8/huy.dinh@emesoft.net"
# }

# Login Docker vào đúng ECR

# aws ecr get-login-password --region ap-southeast-1 --profile AWSDevOpsEngineer-<accountID> \
# | docker login --username AWS --password-stdin <accountID>.dkr.ecr.ap-southeast-1.amazonaws.com
aws ecr get-login-password --region ap-southeast-1 --profile AWSDevOpsEngineer-905418169471 \
  | docker login --username AWS --password-stdin 905418169471.dkr.ecr.ap-southeast-1.amazonaws.com

# Kết quả:
# Login Succeeded

# push image lên ECR
# docker push <accountID>.dkr.ecr.ap-southeast-1.amazonaws.com/<repository_name>:latest
docker push 905418169471.dkr.ecr.ap-southeast-1.amazonaws.com/sabeco-outlet-fe:latest

# Kết quả:
# The push refers to repository [905418169471.dkr.ecr.ap-southeast-1.amazonaws.com/sabeco-outlet-fe]
# 2a263fef437a: Pushed 
# afa2bea6cf14: Pushed 
# cc4da782f375: Pushed 
# 4fae8c5dc10e: Pushed 
# 3b32d9910333: Pushed 
# 418dccb7d85a: Pushed 
# latest: digest: sha256:6daa83908d0bf4ab01f1e7d706a010d17b1387b4b14da29f19998083113d6923 size: 1574

# ec2, pull image về: 
docker pull 905418169471.dkr.ecr.ap-southeast-1.amazonaws.com/sabeco-outlet-fe:latest
 
# dùng docker images kiểm tra
# [ec2-user@ip-172-31-5-120 ~]$ docker images
# REPOSITORY                                                                       TAG                                        IMAGE ID       CREATED         SIZE
# 905418169471.dkr.ecr.ap-southeast-1.amazonaws.com/sabeco-outlet-fe               latest                                     1f6651bbf333   5 hours ago     151MB
# Giả sử image serve web ở port 80 trong container:
docker run -d -p 80:80 --name sabeco-outlet-fe 905418169471.dkr.ecr.ap-southeast-1.amazonaws.com/sabeco-outlet-fe:latest 
