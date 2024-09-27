.PHONY: down up bm sql

down:
	docker compose down

up:
	docker compose up -d

bm:
	cd ../benchmarker && \
	./bin/benchmarker -t "http://localhost" -u ./userdata

sql:
	docker compose exec -it mysql mysql -uroot -proot isuconp