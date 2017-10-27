Extension
=========

Each approach is implemented in a python module in the approaches folder.
It must inherit from :class:`~labelSHARK.core.BaseLabelApproach` and must implement configure and get_labels functions.
It must also include the @LabelSHARK.approach decorator that registers the class with *labelSHARK*.

The configure function is called with a config dict which includes a list of :class:`pycoshark:pycoshark.mongomodels.IssueSystem` under the key **its**.

The set_commit function is called with a :class:`pycoshark:pycoshark.mongomodels.Commit`.
After that the get_labels function is called to request the labels for the commit.

Each approach must also extend the schema for the labelSHARK plugin.


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

        def set_commit(self, commit):
            pass

        def get_labels(self):
            return [('bugfix', True)]

The module of the approach should also be added to /docs/source/approaches.rst so that it shows in the documentation.


Extending the schema for a new approach
---------------------------------------

Take a look at the /plugin_packaging/schema.json. A new approach can add one or multiple fields to the labels for the commit.
The naming is important the approach name which is also the name of the python module should be all lowercase and no dash or underscore.
A new approach detecting bug-fixing commits named findallbugfix would be defined in approaches/findallbugfix.py and add a new field like this:

.. code:: json

    "collections": [
        {
            "collection_name": "commit",
            "fields": [
            {  
               "type":"StructType",
               "logical_type":"Nested",
               "field_name":"labels",
               "desc": "Labels for this commit.",
               "fields": [
                ...
                  {
                     "type":"BooleanType",
                     "logical_type":["CommitLabel", "BugfixLabel"],
                     "field_name":"findallbugfix_bugfix",
                     "desc": "True if this commit is a bug-fixing commit according to the findallbugfix approach, see https://smartshark.github.io/labelSHARK/approaches.html"
                  },
                ...
                ]
        ...



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

