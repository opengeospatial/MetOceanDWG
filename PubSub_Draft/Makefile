all:
	docker run -v "$(PWD)":/metanorma -v ${HOME}/.fontist/fonts/:/config/fonts  metanorma/metanorma  metanorma compile --agree-to-terms -t ogc -x xml,html,doc,pdf document.adoc
clean:
	rm -fr document.doc document.pdf document.*err document.html document.presentation.html document.xml document.xml.abort iev relaton document.presentation.xml document_*_files
