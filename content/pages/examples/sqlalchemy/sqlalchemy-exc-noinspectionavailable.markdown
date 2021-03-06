title: sqlalchemy.exc NoInspectionAvailable code examples
category: page
slug: sqlalchemy-exc-noinspectionavailable-examples
sortorder: 500031003
toc: False
sidebartitle: sqlalchemy.exc NoInspectionAvailable
meta: Python example code for the NoInspectionAvailable class from the sqlalchemy.exc module of the SQLAlchemy project.


NoInspectionAvailable is a class within the sqlalchemy.exc module of the SQLAlchemy project.


## Example 1 from sqlalchemy-utils
[sqlalchemy-utils](https://github.com/kvesteri/sqlalchemy-utils)
([project documentation](https://sqlalchemy-utils.readthedocs.io/en/latest/)
and
[PyPI package information](https://pypi.org/project/SQLAlchemy-Utils/))
is a code library with various helper functions and new data types
that make it easier to use [SQLAlchemy](/sqlachemy.html) when building
projects that involve more specific storage requirements such as
[currency](https://sqlalchemy-utils.readthedocs.io/en/latest/data_types.html#module-sqlalchemy_utils.types.currency).
The wide array of
[data types](https://sqlalchemy-utils.readthedocs.io/en/latest/data_types.html)
includes [ranged values](https://sqlalchemy-utils.readthedocs.io/en/latest/range_data_types.html)
and [aggregated attributes](https://sqlalchemy-utils.readthedocs.io/en/latest/aggregates.html).

[**sqlalchemy-utils / sqlalchemy_utils / functions / foreign_keys.py**](https://github.com/kvesteri/sqlalchemy-utils/blob/master/sqlalchemy_utils/functions/foreign_keys.py)

```python
# foreign_keys.py
from collections import defaultdict
from itertools import groupby

import sqlalchemy as sa
~~from sqlalchemy.exc import NoInspectionAvailable
from sqlalchemy.orm import object_session
from sqlalchemy.schema import ForeignKeyConstraint, MetaData, Table

from ..query_chain import QueryChain
from .database import has_index
from .orm import get_column_key, get_mapper, get_tables


def get_foreign_key_values(fk, obj):
    return dict(
        (
            fk.constraint.columns.values()[index].key,
            getattr(obj, element.column.key)
        )
        for
        index, element
        in
        enumerate(fk.constraint.elements)
    )


def group_foreign_keys(foreign_keys):
    """
    Return a groupby iterator that groups given foreign keys by table.


## ... source file abbreviated to get to NoInspectionAvailable examples ...


        This function does not support exotic mappers that use multiple tables

    .. seealso:: :func:`get_referencing_foreign_keys`
    .. seealso:: :func:`merge_references`

    .. versionadded: 0.26.0
    """
    if foreign_keys is None:
        foreign_keys = get_referencing_foreign_keys(obj)

    session = object_session(obj)

    chain = QueryChain([])
    classes = obj.__class__._decl_class_registry

    for table, keys in group_foreign_keys(foreign_keys):
        keys = list(keys)
        for class_ in classes.values():
            try:
                mapper = sa.inspect(class_)
~~            except NoInspectionAvailable:
                continue
            parent_mapper = mapper.inherits
            if (
                table in mapper.tables and
                not (parent_mapper and table in parent_mapper.tables)
            ):
                query = session.query(class_).filter(
                    sa.or_(*_get_criteria(keys, class_, obj))
                )
                chain.queries.append(query)
    return chain


def _get_criteria(keys, class_, obj):
    criteria = []
    visited_constraints = []
    for key in keys:
        if key.constraint in visited_constraints:
            continue
        visited_constraints.append(key.constraint)

        subcriteria = []
        for index, column in enumerate(key.constraint.columns):
            foreign_column = (


## ... source file continues with no further NoInspectionAvailable examples...


```

