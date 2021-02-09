.PHONY: test
test: vendor check-encoding
	./scripts/test.sh

.PHONY: covhtml
covhtml:
	open .cover/coverage.html

.PHONY: clean
clean:
	git status --ignored --short | grep '^!! ' | sed 's/!! //' | xargs rm -rf

.PHONY: check-encoding
check-encoding:
	! find pkg examples -name "*.go" -type f -exec file "{}" ";" | grep CRLF
	! find scripts -name "*.sh" -type f -exec file "{}" ";" | grep CRLF

.PHONY: fix-encoding
fix-encoding:
	find pkg examples -type f -name "*.go" -exec sed -i -e "s/\r//g" {} +
	find scripts -type f -name "*.sh" -exec sed -i -e "s/\r//g" {} +

.PHONY: vendor
vendor:
	GO111MODULE=on go mod vendor
