Sample usage (assuming you're in cloudbrain root dir):
python cloudbrain/apps/AnalysisModules/AnalysisModulesService.py -i octopicorn -c localhost -n openbci


The basic idea of AnalysisModules is that each module has input/output and the modules are chained in order
with one module's output going into the other's input.  For those of you like horror genre, this can be likened to
"The Human Centipede"

Some assumptions:

- so far, this only works with Pika
- so far this has only been tested with OpenBCI on the "eeg" metric (why would you want anything else?)
- the module processing chain is configured in conf.yml
- each module key "Class" in yaml file must correspond to the exact same class in some file "Class".py at root of
AnalysisModules folder
- If there are more than one modules chained, first module *must* have some input
- each module in yml file has input_feature, output_feature, and parameters. Of these, only input_feature is required
- standard item in parameters per module is "debug".  If True, it governs output to stdout (via print commands)
- modules are loaded, in order, by AnalysisModulesService.py.
- each module is run in its own asynchronous, nonblocking thread
- "metric" and "feature" are interchangeable terms
- Ctrl-C shuts the whole processing chain down, and all the child threads. this is possible because the threads run in
daemon mode, and also because the parent process AnalysisModulesService::start has an endless loop running which
can be interrupted.
- even if you like raw, 99% of the time, the "windows" module is always going to come first in the processing chain.
it makes no sense to update graphs/charts/animations per millisecond. It is performant to update "windows"
or "frames", which are overlapping, in sub 60fps speed.  At the current default settings (500ms window size, 125ms
overlap), the rabbitmq updates windows 14/s. This is better than updating 250 times/sec.

