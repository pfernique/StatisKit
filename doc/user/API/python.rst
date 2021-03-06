.. Copyright [2017-2018] UMR MISTEA INRA, UMR LEPSE INRA,                ..
..                       UMR AGAP CIRAD, EPI Virtual Plants Inria        ..
.. Copyright [2015-2016] UMR AGAP CIRAD, EPI Virtual Plants Inria        ..
..                                                                       ..
.. This file is part of the StatisKit project. More information can be   ..
.. found at                                                              ..
..                                                                       ..
..     http://statiskit.rtfd.io                                          ..
..                                                                       ..
.. The Apache Software Foundation (ASF) licenses this file to you under  ..
.. the Apache License, Version 2.0 (the "License"); you may not use this ..
.. file except in compliance with the License. You should have received  ..
.. a copy of the Apache License, Version 2.0 along with this file; see   ..
.. the file LICENSE. If not, you may obtain a copy of the License at     ..
..                                                                       ..
..     http://www.apache.org/licenses/LICENSE-2.0                        ..
..                                                                       ..
.. Unless required by applicable law or agreed to in writing, software   ..
.. distributed under the License is distributed on an "AS IS" BASIS,     ..
.. WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or       ..
.. mplied. See the License for the specific language governing           ..
.. permissions and limitations under the License.                        ..

The *Python* Interface
======================

The *Python* interface of the **StatisKit** software suite can be installed into a :code:`python-statiskit` environment.
To do so, type the following command lines 

.. code-block:: console

  conda activate
  conda install -n python-statiskit python-statiskit -c statiskit -c defaults --override-channels

Then, to activate the :code:`python-statiskit` environment, type the following command line

.. code-block:: console

  conda activate python-statiskit

.. note::

  If you installed **Conda 2** proceed as follows:

  .. code-block:: console

    conda activate
    conda install -n python-statiskit python=3 -c statiskit -c defaults --override-channels
    conda activate python-statiskit
    conda install python-statiskit -c statiskit -c defaults --override-channels

The *Python* interface of the **StatisKit** software suite can be used as usual for *Python* packages as *Python* scripts that need to be executed, in the *Python* or **IPython** console or from within **Jupyter** notebooks.