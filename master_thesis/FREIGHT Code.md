# Setting
## Config
```C++
bool is_graph_weighted = false;
bool suppress_output = false;
bool recursive = false;  

int ret_code = parse_parameters(argn, argv, 
config,
graph_filename,
is_graph_weighted,
suppress_output, recursive);
```


## Vertex\_Partitioner
```C++
vertex_partitioning* onepass_partitioner = NULL;
initialize_onepass_partitioner(config, onepass_partitioner);
```

```C++
void initialize_onepass_partitioner(PartitionConfig & config, vertex_partitioning*& onepass_partitioner) {
switch(config.one_pass_algorithm) {
	case ONEPASS_HASHING:
	case ONEPASS_HASHING_CRC32:
		onepass_partitioner = new vertex_partitioning(0, config.k-1, config.stream_rec_bisection_base, config.parallel_nodes,config.enable_mapping, config.group_sizes, config.stream_sampling_type, config.stream_sampling, true);
	break;

	case ONEPASS_LDG:
		onepass_partitioner = new onepass_ldg(0, config.k-1, config.stream_rec_bisection_base, config.parallel_nodes, config.enable_mapping, config.group_sizes, config.stream_sampling_type, config.stream_sampling, false);
	break;

	case ONEPASS_FENNEL:

		onepass_partitioner = new onepass_fennel(0, config.k-1, config.stream_rec_bisection_base, config.parallel_nodes,config.enable_mapping, config.group_sizes, config.stream_sampling_type, config.stream_sampling, false, config.fennel_gamma);
	break;

	case ONEPASS_FENNEL_APPROX_SQRT:

	default:
		onepass_partitioner = new onepass_fennel_approx_sqrt(0, config.k-1, config.stream_rec_bisection_base, config.parallel_nodes, config.enable_mapping, config.group_sizes, config.stream_sampling_type, config.stream_sampling, false, config.fennel_gamma);
	break;
}

if (config.use_self_sorting_array) onepass_partitioner->enable_self_sorting_array();
onepass_partitioner->set_sampling_threashold(config.sampling_threashold);
}
```

所以
- `onepass_partitioner` 是 `onepass_fennel_approx_sqrt(0, config.k-1, config.stream_rec_bisection_base, config.parallel_nodes, config.enable_mapping, config.group_sizes, config.stream_sampling_type, config.stream_sampling, false, config.fennel_gamma);`

`onepas_fennel_approx_sqrt` 初始化如下
```C++
onepass_fennel_approx_sqrt(PartitionID k0, PartitionID kf, PartitionID max_blocks, NodeID n_threads, bool given_hierarchy, std::vector<int> & group_sizes, int sampling_type, PartitionID n_samples=0, bool hashing=false, float gamma=1.5);
```

```C++
typedef enum {
	SAMPLING_INACTIVE,
	SAMPLING_NEIGHBORS,
	SAMPLING_NONNEIGHBORS,
	SAMPLING_BLOCKS,
	SAMPLING_TWOFOLD,
	SAMPLING_INACTIVE_LINEAR_COMPLEXITY,
	SAMPLING_NEIGHBORS_LINEAR_COMPLEXITY
} SamplingType;
```

用`printf`大法，知道參數如下
- `config.stream_sampling_type` is: 5, 依據上面的`enum`知 `sampling_type` 是 `SAMPLING_INACTIVE_LINEAR_COMPLEXITY`
- `config.stream_sampling` is: 0; 
- `onepass_partitioner` is `ONEPASS_FENNEL_APPROX_SQRT` : 1 $\Rightarrow$ 採用的分割函數是`ONEPASS_FENNEL_APPROX_SQRT`

```C++
inline PartitionID vertex_partitioning::solve_node(LongNodeID curr_node_id, NodeWeight curr_node_weight, int my_thread) {
	PartitionID decision;
	if (hashing) {
		decision = solve_hashing(curr_node_id, curr_node_weight, my_thread);

	} else {
		this->amortized_rounds_for_feasibility_sampling++;
		switch(sampling) {
			case SAMPLING_INACTIVE_LINEAR_COMPLEXITY:
				decision = solve_linear_complexity(curr_node_id, curr_node_weight, my_thread);

#pragma omp critical(update_self_sorting_vector)

this->sorted_blocks.increment(decision);

break;

case SAMPLING_NEIGHBORS_LINEAR_COMPLEXITY:

decision = solve_sampl_neighb_linear_complex(curr_node_id, curr_node_weight, my_thread);

#pragma omp critical(update_self_sorting_vector)

this->sorted_blocks.increment(decision);

break;

case SAMPLING_INACTIVE:

decision = solve(curr_node_id, curr_node_weight, my_thread);

break;

case SAMPLING_NEIGHBORS:

decision = solve_sampling_neighbors(curr_node_id, curr_node_weight, my_thread);

break;

case SAMPLING_NONNEIGHBORS:

decision = solve_sampling_nonneighbors(curr_node_id, curr_node_weight, my_thread);

break;

case SAMPLING_BLOCKS:

decision = solve_sampling_blocks(curr_node_id, curr_node_weight, my_thread);

break;

case SAMPLING_TWOFOLD:

decision = solve_sampling_twofold(curr_node_id, curr_node_weight, my_thread);

break;

}

}

return decision;

}
```

因為 `sampling` 是指 `SAMPLING_INACTIVE_LINEAR_COMPLEXITY` 所以 `solve_node` 是用 `solve_linear_complexity`

```C++
inline PartitionID vertex_partitioning::solve_linear_complexity(LongNodeID curr_node_id, NodeWeight curr_node_weight, int my_thread) {
	float best = std::numeric_limits<float>::lowest();
	PartitionID decision = random_functions::nextIntHashing(blocks.size());
	/* NodeID decision = crc32(curr_node_id)% (blocks.size()); */
	gothrough_neighborhood(decision, best, my_thread);
	check_best_nonneighbor(decision, best, my_thread);
	/* amortized_sampling_for_feasibility(decision); */
	return set_decision(decision, curr_node_id, curr_node_weight, my_thread);
}
```

```C++
inline void vertex_partitioning::gothrough_neighborhood(NodeID& decision, float& best, int my_thread) {
	float score;
	for (auto id : neighbor_blocks[my_thread]) {
		auto& block = blocks[id];
		if (block.fully_loaded()) {
			continue;
		}
		score = compute_score(block, my_thread);
		if (score > best) {
			decision = block.get_block_id();
			best = score;
		}
	}
}
```


```C++
inline void vertex_partitioning::check_best_nonneighbor(NodeID& decision, float& best, int my_thread) {
	float score;
	PartitionID id = this->sorted_blocks[0]; // it does not matter whether or not it is a nonneighbor
	auto& block = blocks[id];
	score = compute_score(block, my_thread);
	if (score > best) {
		decision = block.get_block_id();
		best = score;
	}
}
```


```C++
template <typename K, typename V>
inline void self_sorting_monotonic_vector<K,V>::initialize(K n_elements, V initial_value) {
	this->n_elements = n_elements;
	BUCKET<K,V> my_bucket = {this->n_elements, initial_value, this->n_elements - 1};

	this->bucket_list.push_front(my_bucket);
	this->ordered_list.resize(this->n_elements);
	this->pos.resize(this->n_elements);
	for (K i=(K)0; i<this->n_elements; i++) {
		this->ordered_list[i].key = i;
		this->ordered_list[i].buck = this->bucket_list.begin();
		this->pos[i] = i;
	}
}
```
  
```C++
template <typename K, typename V>
inline void self_sorting_monotonic_vector<K,V>::increment(K key) {

	V new_value = get_value(key) + 1;
	remove_from_bucket(key);

	if (next_bucket_matches_value(key, new_value)) {
		assign_to_next_bucket(key);
	} 
	else {
		assign_to_new_bucket(key, new_value);
	}
}
```

# Output
```c++
	// write the partition to the disc
    std::stringstream filename;
    if(!config.filename_output.compare("")) {
        filename << "tmppartition" << extract_name(argv[0], argv[1]) << config.k << "_imb_" << config.imbalance;
    } 
    else {
        filename << config.filename_output;
    }

    if (!config.suppress_file_output) {
        graph_io_stream::writePartitionStream(config, filename.str());
    } 
    else {
        std::cout << "No partition will be written as output." << std::endl;
    }
```

**關鍵在於 `graph_io_stream::writePartitionStream()` 是否按順序輸出**
```c++
void graph_io_stream::writePartitionStream(PartitionConfig & config, const std::string & filename) {
        std::ofstream f(filename.c_str());
        std::cout << "writing partition to " << filename << " ... " << std::endl;
        for (int node = 0; node < config.stream_nodes_assign->size(); node++) {
                f << (*config.stream_nodes_assign)[node] <<  "\n";
        }
        f.close();
}
```
