# Minimal makefile for Sphinx documentation
#

# You can set these variables from the command line.
SPHINXOPTS    =
SPHINXBUILD   = sphinx-build
SPHINXPROJ    = clean_code
SOURCEDIR     = .
BUILDDIR      = _build

# Put it first so that "make" without argument is like "make help".
help:
	@$(SPHINXBUILD) -M help "$(SOURCEDIR)" "$(BUILDDIR)" $(SPHINXOPTS) $(O)

.PHONY: help Makefile

# Catch-all target: route all unknown targets to Sphinx using the new
# "make mode" option.  $(O) is meant as a shortcut for $(SPHINXOPTS).
%: Makefile
	# sphinx-apidoc -M -f -o modules ../clean_code
	# rm modules/modules.rst
	# for f in modules/clean_code.*.rst ; do \
	#     sed -i "1,10d" $$f ; \
	# done
	@$(SPHINXBUILD) -M $@ "$(SOURCEDIR)" "$(BUILDDIR)" $(SPHINXOPTS) $(O)
	@if [ $@ = "html" ]; then \
    	rm -rf ../docs; \
    	cp -r _build/html ../docs; \
    	touch ../docs/.nojekyll; \
    	rm -rf _build; \
    else \
    	rm -rf ../dpt.pdf; \
    	cp _build/latex/clean_code.pdf ../clean_code.pdf; \
    	rm -rf _build; \
    fi
