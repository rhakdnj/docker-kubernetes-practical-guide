FROM node

WORKDIR /app

COPY . .

RUN npm install

# 포트를 명시했다고 달라지는건 없다.
# 'docker run -p' 옵션으로 포트를 명시하지 않으면 컨테이너의 포트가 호스트 운영체제에 공개되지 않는다.
# EXPOSE 구문으로 명시한 포트는 'docker run -P' 명령을 이용할 때 호스트 운영체제로 오픈됩니다.
# 이 때, 호스트 운영체제의 랜덤 포트 번호가 컨테이너의 EXPOSE 구문으로 명시한 포트에 매핑됩니다.
EXPOSE 80

CMD ["node", "server.js"]
