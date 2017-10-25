Extension
=========

Each approach is implemented in a python module in the approaches folder.
It must inherit from :class:`~labelSHARK.core.BaseLabelApproach` and must implement configure and get_labels functions.
It should also include the @LabelSHARK.approach decorator that registers the class with *labelSHARK*.

The configure function is called with a config dict which includes a list of :class:`pycoshark:pycoshark.mongomodels.IssueSystem` under the key **its**.

The get_labels function is called with a :class:`pycoshark:pycoshark.mongomodels.Commit` until all commits for the examined projects VCS are labeled.


Minimum Example
---------------

Below is the code for a minimum working example which labels every commit as bugfix commit.
The code should be placed in the approaches directory in a python file which name is the approach itself, e.g., approaches/szz.py.

The get_labels method returns a list of tuples which get prefixed by the name of the module of the approach, e.g., szz, as described above. The approach can return multiple tuples for multiple labels, e.g., [('corrective', True),('maintenance', False)].

.. code::

    #!/usr/bin/env python
    # -*- coding: utf-8 -*-

    # needed imports
    from core import LabelSHARK, BaseLabelApproach

    # SmartSHARK mongo models if needed
    from pycoshark.mongomodels import Issue, Event


    @LabelSHARK.approach
    class MinimumApproach(BaseLabelApproach):
        """Short description of the approach.

        Long description of the approach.
        """

        def configure(self, config):
            pass

        def get_labels(self, commit):
            return [('bugfix', True)]

The module of the approach should also be added to /docs/source/approaches.rst so that it shows in the documentation.


Special case: missing index rights in MongDB
--------------------------------------------

If the user connecting to the MongoDB has no index rights, working with the models may fail.
This workaround deletes the index from the meta information of the models:

.. code::

    import copy
    from pycoshark.mongomodels import Issue, Event

    def remove_index(cls):
        tmp = copy.deepcopy(cls._meta)
        if 'indexes' in tmp.keys():
            del tmp['indexes']
        del tmp['index_specs']
        tmp['index_specs'] = None
        return tmp

    Issue._meta = remove_index(Issue)
    Event._meta = remove_index(Event)

