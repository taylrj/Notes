# Code Splitting

## Webpack HashedModuleIdsPlugin



## Webpack CommonsChunkPlugin

### Options

- webpack official documentation
  
```js
{
  name: string, // or
  names: string[],
  // The chunk name of the commons chunk. An existing chunk can be selected by passing a name of an existing chunk.
  // If an array of strings is passed this is equal to invoking the plugin multiple times for each chunk name.
  // If omitted and `options.async` or `options.children` is set all chunks are used, otherwise `options.filename`
  // is used as chunk name.
  // When using `options.async` to create common chunks from other async chunks you must specify an entry-point
  // chunk name here instead of omitting the `option.name`.

  filename: string,
  // The filename template for the commons chunk. Can contain the same placeholders as `output.filename`.
  // If omitted the original filename is not modified (usually `output.filename` or `output.chunkFilename`).
  // This option is not permitted if you're using `options.async` as well, see below for more details.

  minChunks: number|Infinity|function(module, count) => boolean,
  // The minimum number of chunks which need to contain a module before it's moved into the commons chunk.
  // The number must be greater than or equal 2 and lower than or equal to the number of chunks.
  // Passing `Infinity` just creates the commons chunk, but moves no modules into it.
  // By providing a `function` you can add custom logic. (Defaults to the number of chunks)

  chunks: string[],
  // Select the source chunks by chunk names. The chunk must be a child of the commons chunk.
  // If omitted all entry chunks are selected.

  children: boolean,
  // If `true` all children of the commons chunk are selected

  deepChildren: boolean,
  // If `true` all descendants of the commons chunk are selected

  async: boolean|string,
  // If `true` a new async commons chunk is created as child of `options.name` and sibling of `options.chunks`.
  // It is loaded in parallel with `options.chunks`.
  // Instead of using `option.filename`, it is possible to change the name of the output file by providing
  // the desired string here instead of `true`.

  minSize: number,
  // Minimum size of all common module before a commons chunk is created.
}
```
