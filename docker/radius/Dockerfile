FROM freeradius/freeradius-server:latest

# 커스텀 설정 파일 복사
COPY raddb/users    /etc/raddb/users
COPY raddb/clients.conf /etc/raddb/clients.conf

# 개발·디버그 모드로 실행 (검증 후 -X 제거 가능)
CMD ["radiusd", "-X"]
# test2
