DOT_OPTIONS = -Tsvg -Nfontname=Helvetica -Gfontname=Helvetica -Nfontsize=10 -Gfontsize=10

all: init-term.dot.svg init-term.dot.2.svg

init-term.dot.svg: init-term.dot
	dot $(DOT_OPTIONS) -O $<
	sed -i .orig -e 's/<title>[^<]*<\/title>//g' init-term.*.svg

init-term.dot.2.svg: init-term.dot.svg
	touch $@
