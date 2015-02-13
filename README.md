# PlanOut

PlanOut is a framework and programming language for online field experimentation. PlanOut was created to make it easy to run and iterate on sophisticated experiments, while satisfying the constraints of deployed Internet services with many users.

Developers integrate PlanOut by defining experiments that detail how _units_ (e.g., users, cookie IDs) should get mapped onto conditions. For example, to create a 2x2 experiment randomizing both the color and the text on a button, you create a class like this in Python:

```python
class MyExperiment(SimpleExperiment):
  def assign(self, params, userid):
    params.button_color = UniformChoice(choices=['#ff0000', '#00ff00'], unit=userid)
    params.button_text = UniformChoice(choices=['I voted', 'I am a voter'], unit=userid)
```

Then, in the application code, you query the Experiment object to find out what values the current user should be mapped onto:
```python
my_exp = MyExperiment(userid=101)
color = my_exp.get('button_color')
text = my_exp.get('button_text')
```

PlanOut takes care of randomizing each ``userid`` into the right bucket. It does so by hashing the input, so each ``userid`` will always map onto the same values for that experiment.

### What does the PlanOut distribution include?

The reference implementation for PlanOut is written in Python.  It includes:
  * Extensible classes for defining experiments. These classes make it easy to implement reliable, deterministic random assignment procedures, and automatically log key data.
  * A basic implementation of namespaces, which can be used to manage multiple mutually exclusive experiments.
  * The PlanOut interpreter, which executes serialized code generated by the PlanOut domain specific language.
  * An interactive Web-based editor and compiler for developing and testing
  PlanOut-language scripts.

A production-ready version of PlanOut is also available for PHP, and can found in the `php/` directory.

The `alpha/` directory contains implementations of PlanOut to other languages. Hack and Go-based implementations will be available in early 2015.

### Who is PlanOut for?
PlanOut designed for researchers, students, and small businesses wanting to run experiments. It is built to be extensible, so that it may be adapted for use with large production environments.  The implementation here mirrors many of the key components of Facebook's Hack-based implementation of PlanOut which is used to conduct experiments with hundreds of millions of users.

### Full Example

To create a basic PlanOut experiment, you subclass ``SimpleExperiment`` object, and implement an assignment method. You can use PlanOut's random assignment operators by setting ``e.varname``, where ``params`` is the first argument passed to the ``assign()`` method, and ``varname`` is the name of the variable you are setting.
```python

from planout.experiment import SimpleExperiment
from planout.ops.random import *

class FirstExperiment(SimpleExperiment):
  def assign(self, params, userid):
    params.button_color = UniformChoice(choices=['#ff0000', '#00ff00'], unit=userid)
    params.button_text = WeightedChoice(
        choices=['Join now!', 'Sign up.'],
        weights=[0.3, 0.7], unit=userid)

my_exp = FirstExperiment(userid=12)
# parameters may be accessed via the . operator
print my_exp.get('button_text'), my_exp.get('button_color')

# experiment objects include all input data
for i in xrange(6):
  print FirstExperiment(userid=i)
```

which outputs:
```
Join now! #ff0000
{'inputs': {'userid': 0}, 'checksum': '22c13b16', 'salt': 'FirstExperiment', 'name': 'FirstExperiment', 'params': {'button_color': '#ff0000', 'button_text': 'Sign up.'}}
{'inputs': {'userid': 1}, 'checksum': '22c13b16', 'salt': 'FirstExperiment', 'name': 'FirstExperiment', 'params': {'button_color': '#ff0000', 'button_text': 'Sign up.'}}
{'inputs': {'userid': 2}, 'checksum': '22c13b16', 'salt': 'FirstExperiment', 'name': 'FirstExperiment', 'params': {'button_color': '#00ff00', 'button_text': 'Sign up.'}}
{'inputs': {'userid': 3}, 'checksum': '22c13b16', 'salt': 'FirstExperiment', 'name': 'FirstExperiment', 'params': {'button_color': '#ff0000', 'button_text': 'Sign up.'}}
{'inputs': {'userid': 4}, 'checksum': '22c13b16', 'salt': 'FirstExperiment', 'name': 'FirstExperiment', 'params': {'button_color': '#00ff00', 'button_text': 'Join now!'}}
{'inputs': {'userid': 5}, 'checksum': '22c13b16', 'salt': 'FirstExperiment', 'name': 'FirstExperiment', 'params': {'button_color': '#00ff00', 'button_text': 'Sign up.'}}
```

The ``SimpleExperiment`` class will automatically concatenate the name of the experiment, ``FirstExperiment``, the variable name, and the input data (``userid``) and hash that string to perform the random assignment. Parameter assignments and inputs are automatically logged into a file called ``firstexperiment.log'``.

For more information on using PlanOut with PHP, see the README for Vimeo's [port of PlanOut to PHP](https://github.com/vimeo/ABLincoln), ABLincoln.

### Installation
You can immediately install PlanOut using `pip` with:
```
pip install planout
```

Or you can checkout the Github repository and type:

```
sudo python setup.py install
```

### Learn more
Learn more about PlanOut visiting the [PlanOut website](http://facebook.github.io/planout/) or by [reading the PlanOut paper](http://arxiv.org/pdf/1409.3174v1.pdf). You can cite PlanOut as "Designing and Deploying Online Field Experiments". Eytan Bakshy, Dean Eckles, Michael S. Bernstein. Proceedings of the 23rd ACM conference on the World Wide Web. April 7–11, 2014, Seoul, Korea, or by copying and pasting the bibtex below:
``` bibtex
@inproceedings{bakshy2014www,
	Author = {Bakshy, E. and Eckles, D. and Bernstein, M.S.},
	Booktitle = {Proceedings of the 23rd ACM conference on the World Wide Web},
	Organization = {ACM},
	Title = {Designing and Deploying Online Field Experiments},
	Year = {2014}
}
```
