# Parallel JPEG XS Implementation

This repository contains a parallel implementation of the JPEG XS image compression standard, based on the reference implementation. The parallelization focuses on the precinct processing loops in both encoder and decoder, utilizing OpenMP for multi-threading support.

## Key Features

- OpenMP-based parallel processing of image precincts
- Thread-safe bitstream handling
- Atomic operations for shared resource access
- Dynamic scheduling for load balancing
- Maintains standards compliance with ISO/IEC 21122

## Implementation Details

### Encoder Parallelization

The encoder's main precinct processing loop has been parallelized using OpenMP:
```c
#pragma omp parallel for schedule(dynamic) private(rc_results) shared(ctx, image, slice_idx, markers_len)
for (int line_idx = 0; line_idx < image->height; line_idx += ctx->ids.ph) {
    // Process precincts with thread safety
}
```

Key features:
- Parallel processing of image lines
- Dynamic scheduling for better load balancing
- Critical sections protect shared resources:
  - Bitstream writing
  - Rate control updates
  - Slice header management
- Thread-private rate control results

### Decoder Parallelization

The decoder's precinct processing loop features similar parallelization:
```c
#pragma omp parallel for schedule(dynamic) private(unpack_out) shared(ctx, image_out, slice_idx, bitstream_pos)
for (int line_idx = 0; line_idx < ctx->ids.h; line_idx += ctx->ids.ph) {
    // Process precincts with thread safety
}
```

Key features:
- Parallel processing of image lines
- Thread-safe bitstream unpacking
- Protected access to shared precinct buffers
- Atomic updates for bitstream position tracking
- Safe handling of slice headers

## Build Requirements

- CMake 3.12 or newer
- C compiler with OpenMP support (GCC 9.3+ recommended)
- OpenMP development libraries

## Building

```bash
mkdir build
cd build
cmake ..
make
```

## Performance Considerations

The parallel implementation offers improved performance through:
- Multi-threaded precinct processing
- Dynamic load balancing
- Minimal thread synchronization overhead
- Efficient memory access patterns

Note: Actual performance gains depend on:
- Image size and precinct dimensions
- Available CPU cores
- Memory bandwidth
- I/O subsystem capabilities

## Thread Safety

The implementation ensures thread safety through:
1. Critical sections around shared resource access
2. Atomic operations for counters and flags
3. Thread-private copies of key data structures
4. Careful management of bitstream access
5. Protected slice header processing

## Usage Example

The parallel implementation maintains the same API as the reference implementation:

```c
// Initialize encoder/decoder with default thread count
xs_enc_context_t* ctx = xs_enc_init(config, image);

// Optionally set thread count (default: max available)
omp_set_num_threads(thread_count);

// Process image
xs_enc_image(ctx, image, buffer, buffer_size, &output_size);
```

## Limitations

- Some operations remain serial due to data dependencies
- Bitstream access requires synchronization
- Very small images may not benefit from parallelization
- Memory bandwidth can become a bottleneck

## License

This implementation maintains the original JPEG XS license terms. See the license header in source files for details.
