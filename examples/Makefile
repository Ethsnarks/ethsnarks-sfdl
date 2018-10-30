SFDL1_PROGS=$(wildcard *.sfdl)

OPT_CIRCUITS=$(addsuffix .Opt.circuit,$(SFDL1_PROGS))
NOOPT_CIRCUITS=$(addsuffix .NoOpt.circuit,$(SFDL1_PROGS))

all: $(OPT_CIRCUITS) $(NOOPT_CIRCUITS)

%.Opt.circuit: %
	java -jar ../dist/sfdl.jar $<

%.NoOpt.circuit: %
	java -jar ../dist/sfdl.jar -no-opt $<

clean:
	rm -f *.fmt *.circuit
