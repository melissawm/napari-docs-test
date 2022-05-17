(readers-contribution-guide)=
## Readers

Reader plugins may add support for new filetypes to napari.
They are invoked whenever `viewer.open('some/path')` is used on the
command line, or when a user opens a file in the graphical user interface by
dropping a file into the canvas, or using `File -> Open...`

The `command` provided by a reader contribution is expected to be a function
that accepts a path (`str`) or a list of paths and:
* returns `None` (if it does not want to accept the given path)
* returns a *new function* (a `ReaderFunction`) that is capable of doing the reading.

The `ReaderFunction` will be passed the same path (or list of paths) and
is expected to return a list of {ref}`LayerData tuples <layer-data-tuples>`.

In the rare case that a reader plugin would like to "claim" a file, but *not*
actually add any data to the viewer, the `ReaderFunction` may return
the special value `[(None,)]`.

```{admonition} Accepting directories
A reader may indicate that it accepts directories by
setting `contributions.readers.<reader>.accepts_directories` to `True`;
otherwise, they will not be invoked when a directory is passed to `viewer.open`.
```

### Reader example

````{tabbed} npe2
**python implementation**

```python
# example_plugin.some_module
PathLike = str
PathOrPaths = Union[PathLike, Sequence[PathLike]]
def _ensure_str_or_seq_str(path):
    if isinstance(path, Path) or (
        isinstance(path, (list, tuple)) and any([isinstance(p, Path) for p in path])
    ):
        warnings.warn(
            "Npe2 receive a `Path` or a list of `Path`s, instead of `str`,"
            " this will  become an error in the future and is likely a"
            " napari bug. Please fill and issue.",
            UserWarning,
            stacklevel=3,
        )

ReaderFunction = Callable[[PathOrPaths], List[LayerData]]


def get_reader(path: PathOrPaths) -> Optional[ReaderFunction]:
    # If we recognize the format, we return the actual reader function
    if isinstance(path, str) and path.endswith(".xyz"):
        return xyz_file_reader
    # otherwise we return None.
    return None


def xyz_file_reader(path: PathOrPaths) -> List[LayerData]:
    data = ...  # somehow read data from path
    layer_attributes = {"name": "etc..."}
    return [(data, layer_attributes)]
```

**manifest**

See [Readers contribution reference](./contributions.html#contributions-readers)
for field details.

```yaml
contributions:
  commands:
  - id: example-plugin.read_xyz
    title: Read ".xyz" files
    python_name: example_plugin.some_module:get_reader
  readers:
  - command: example-plugin.read_xyz
    filename_patterns:
    - '*.xyz'
    accepts_directories: false

```
````

````{tabbed} napari-plugin-engine

```{admonition} Deprecated!
This demonstrates the now-deprecated `napari-plugin-engine` pattern.
```

**python implementation**

[hook specification](https://napari.org/plugins/stable/npe1.html#napari.plugins.hook_specifications.napari_get_reader)

```python
from napari_plugin_engine import napari_hook_implementation


@napari_hook_implementation
def napari_get_reader(path: PathOrPaths) -> Optional[ReaderFunction]:
    # If we recognize the format, we return the actual reader function
    if isinstance(path, str) and path.endswith(".xyz"):
        return xyz_file_reader
    # otherwise we return None.
    return None


def xyz_file_reader(path: PathOrPaths) -> List[LayerData]:
    data = ...  # somehow read data from path
    layer_properties = {"name": "etc..."}
    return [(data, layer_properties)]
```
````