# Tagged Format

The Tagged Format is a simple text and binary format designed to be loaded directly into the memory of the running process. It uses a simple text format that can be incrementally parsed and serialized, and the binary format uses tagged memory blocks and offsets for addressing.

It is primarily used for storing 3D model data, and includes an exporter for [Blender](https://www.blender.org/) and both a C++ and JavaScript implementation. However, it's design is generic and it can be modified to suit a wide range of tasks.

[![Build Status](https://secure.travis-ci.org/kurocha/tagged-format.svg)](http://travis-ci.org/kurocha/tagged-format)

## Build and Install

Use Teapot to build and install Tagged Format:

	$ cd tagged-format
	$ bundle install 
	$ bundle exec teapot fetch
	$ bundle exec teapot build Library/TaggedFormat variant-debug

## File Format

The Tagged file format is designed to be application specific rather than generic. Many generic file formats already exist (e.g. Wavefront OBJ) but they are cumbersome to use because they lack the ability to be customised without a significant amount of work, or require significant work at run-time.

Rather than try to design a one-size fits all general file format, you are encouraged to fork the library on a per-application basis and build specific resource formats that match your exact needs. Cases where you  might want to customise the behavior include unusual vertex formats (e.g. multiple texture coordinates, additional per-vertex attributes, etc) or embedded binary data (e.g. animation data, texture data, shaders).

The tagged binary format consists of a sequential set of blocks, where each block has a tag and size. The basic file has a header which includes an offset to the top block. By default, a variety of block types are included in the specification, including a `Table`, `Mesh`, `Axis`, a variety of vertex formats and `External` references.

The tagged text format is well defined and is compiled to a binary format using the included `TaggedFormat`, much like how an assembler converts symbolic code to binary machine code.

### Matrices

Matrices are represented in all cases in the order they will be in-memory. When writing this in text, the matrix is therefore listed in rows.

## Tagged Format Tool

Included in this implementation is the `TaggedFormat` executable which converts a text format into a binary format. The text format is primarily used as an export format and is typically converted into the binary format before being used.

Here is an example of a simple 10x10 square made by two triangles:

	top: mesh triangles
		indices: array index16
			0 1 2 3
		end

		vertices: array vertex-p3n3m2
			 0  0 0 0 0 1 0 0
			 0 10 0 0 0 1 0 1
			10  0 0 0 0 1 1 0
			10 10 0 0 0 1 1 1
		end
	end

To assemble this to the binary TMF format, simply run:

	TaggedFormat --text-to-binary input.tft output.tfb

You can check the results by running:

	$ TaggedFormat --dump-binary output.tfb
	<tmv2; 32 bytes; magic = 42>
	[mesh; 48 bytes; offset = 32]
		layout = triangles
		indices:
		[ind2; 24 bytes; offset = 80]
			0 1 2 3 
		vertices:
		[3320; 144 bytes; offset = 104]
			P=(0, 0, 0) N=(0, 0, 1) M=(0, 0)
			P=(0, 10, 0) N=(0, 0, 1) M=(0, 1)
			P=(10, 0, 0) N=(0, 0, 1) M=(1, 0)
			P=(10, 10, 0) N=(0, 0, 1) M=(1, 1)
		axes:
		[null: 0 bytes]

## Rendering

Loading the file is very simple and fast as data can be loaded almost directly into graphics memory. Use the `TaggedFormat::Reader` to load a model from a data buffer (typically loaded from disk using `mmap`).

	Model model = load_model(state->resource_loader, "base/models/baseplate.tfb");

	// Read header:
	auto header = model.reader.header();
	auto dictionary_block = model.reader.block_at_offset<TaggedFormat::Table>(header->top_offset);

	auto offset = dictionary_block->offset_named("baseplate");
	auto mesh_block = model.reader.block_at_offset<TaggedFormat::Mesh>(offset);

	// mesh_block provides data offsets for indices, vertices and other mesh-related data:
	_mesh = load_mesh(model, mesh_block);
	
	_mesh->render();

The Dream framework includes an example of how to load and render models using the tagged model format.

## Caveats

The binary format is platform specific. It would be possible to adjust data ordering (e.g. endian) however this goes against the spirit of the Tagged binary format: a simple loader that provides platform and application specific data that can be quickly loaded into the graphics card.

In the case where you need some form of platform independence, there are two options: Either integrate the `tagged-format-convert` into your application development toolchain to produce target-specific binary versions of your model data, or integrate the parser directly into your application and cache binary versions of the model data as required.

## Future Work

- Provide target specific options to `tagged-format-convert` for generating platform specific (e.g. endian) binary data.
- Develop a simple Cocoa application for visualising the model data structure and rendering meshes.
- Complete animation related data-structures and provide examples.

## License

Released under the MIT license.

Copyright, 2016, by [Samuel G. D. Williams](http://www.codeotaku.com/samuel-williams).

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
