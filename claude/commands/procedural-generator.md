# Procedural Generation System Designer

You are a procedural generation engine implementing the Watson et al. (2008) workflow. Given a topic, you execute each phase sequentially, generating concrete outputs at each step. Use classical algorithms only — NO LLM integration.

## Input

`$ARGUMENTS` should contain the topic/subject to generate a procedural system for. If not provided, ask the user.

**TOPIC**: `$ARGUMENTS`

## Execution Pipeline

### PHASE 1: Design Analysis & Parameterization

**Objective:** Decompose the topic into hierarchical components and quantifiable parameters.

Execute:
```pseudocode
FUNCTION analyze_design(topic):
    // Hierarchical decomposition
    primary_elements = extract_core_components(topic)
    secondary_elements = extract_detail_components(topic)

    // Parameter identification
    FOR each element IN primary_elements:
        geometric_params = {scale, proportion, orientation, position}
        aesthetic_params = {color, texture, material, style}
        behavioral_params = {frequency, variation, constraints}

    // Constraint analysis
    spatial_constraints = define_spatial_relationships()
    aesthetic_constraints = define_style_consistency()

    RETURN {elements, parameters, constraints}
```

**Output Requirements:**
- Element taxonomy: `E = {e1, e2, ..., en}` where each `ei` has attributes `{geometry, material, behavior}`
- Parameter space: `P = {p1 in [min1, max1], p2 in [min2, max2], ..., pm in [minm, maxm]}`
- Constraint matrix: `C[i,j] = relationship_strength(ei, ej)`

**Debug Output:** Print element hierarchy tree, parameter ranges, constraint violations.

---

### PHASE 2: Primitive Creation & Texture Synthesis

**Objective:** Generate reusable building blocks and surface materials.

Execute:
```pseudocode
FUNCTION create_primitives(elements, parameters):
    primitive_library = {}

    FOR each element IN elements:
        // Geometric primitive generation
        base_geometry = generate_base_mesh(element.geometry)
        variations = apply_parameter_variations(base_geometry, parameters)

        // Texture synthesis using classical methods
        IF element.material == "procedural":
            texture = perlin_noise_synthesis(element.texture_params)
        ELIF element.material == "tiled":
            texture = wang_tile_synthesis(element.pattern)
        ELSE:
            texture = cellular_automata_synthesis(element.rules)

        primitive_library[element.id] = {geometry: variations, texture: texture}

    RETURN primitive_library
```

**Mathematical Foundation:**
- Perlin noise: `N(x,y) = Sum_i a_i * noise(2^i * x, 2^i * y)` where `a_i = persistence^i`
- Wang tiles: `T(x,y) = tile_lookup[hash(floor(x/tile_size), floor(y/tile_size)) % tile_count]`
- Cellular automata: `C(t+1) = rule_function(neighborhood(C(t)))`

**Output Requirements:**
- Primitive mesh library with LOD variants
- Tileable texture atlas with seamless boundaries
- Material property definitions (roughness, metallic, emission)

**Debug Output:** Mesh statistics, texture memory usage, tiling verification.

---

### PHASE 3: Grammar Rule Encoding

**Objective:** Define shape grammar for hierarchical assembly.

Execute:
```pseudocode
FUNCTION encode_grammar_rules(elements, constraints):
    grammar = ShapeGrammar()

    // Production rules: A -> BC (split), A -> B (substitute), A -> e (terminate)
    FOR each element IN elements:
        IF element.type == "container":
            rule = SplitRule(element, split_direction, split_ratios)
        ELIF element.type == "decorator":
            rule = SubstituteRule(element, replacement_probability)
        ELSE:
            rule = TerminalRule(element, geometry_instance)

        grammar.add_rule(rule)

    // Context-sensitive constraints
    FOR each constraint IN constraints:
        grammar.add_context_rule(constraint.condition, constraint.action)

    RETURN grammar
```

**Grammar Formalism:**
- Production rule: `R: L -> R` where L is left-hand side, R is right-hand side
- Split rule: `Split(axis, ratios) : A -> B1(r1) | B2(r2) | ... | Bn(rn)` where `Sum(ri) = 1`
- Context rule: `C: A -> B [condition(neighborhood)]`

**Rule Examples:**
```
Building -> Ground_Floor | Middle_Floors* | Top_Floor
Facade -> Window_Pattern(0.7) | Wall_Pattern(0.3)
Window_Pattern -> Window{repeat(3,8)} | Wall{remainder}
```

**Output Requirements:**
- Complete rule set with precedence ordering
- Context-sensitive constraint definitions
- Rule application probability distributions

**Debug Output:** Rule dependency graph, circular reference detection, coverage analysis.

---

### PHASE 4: Stochastic Behavior Integration

**Objective:** Add controlled randomness for variation without chaos.

Execute:
```pseudocode
FUNCTION add_stochastic_behavior(grammar, parameters):
    stochastic_grammar = grammar.copy()

    FOR each rule IN grammar.rules:
        // Parameter randomization
        FOR each param IN rule.parameters:
            if param.allow_variation:
                noise_func = select_noise_function(param.variation_type)
                param.value = param.base_value + noise_func(param.seed) * param.variation_range

        // Rule selection probabilities
        IF rule.has_alternatives:
            probabilities = normalize(rule.alternative_weights + random_perturbation())
            rule.selection_probability = probabilities

    // Spatial correlation for coherent variation
    spatial_seed_map = generate_spatial_seeds(world_bounds, correlation_distance)

    RETURN {stochastic_grammar, spatial_seed_map}
```

**Stochastic Methods:**
- Uniform distribution: `U(a,b) = a + (b-a) * rand()`
- Normal distribution: `N(mu,sigma) = mu + sigma * box_muller_transform(rand1, rand2)`
- Spatial correlation: `corr(p1,p2) = exp(-||p1-p2||^2 / 2*sigma^2)`

**Variation Control:**
- Global coherence: `variation_strength = base_strength * coherence_field(position)`
- Local constraints: `clamp(value, local_min, local_max)`
- Temporal stability: `seed = hash(position) XOR global_seed`

**Output Requirements:**
- Parameterized probability distributions
- Spatial correlation functions
- Seed management system for reproducibility

**Debug Output:** Statistical distribution plots, correlation heatmaps, seed collision detection.

---

### PHASE 5: Model Generation & Instantiation

**Objective:** Execute grammar rules to produce final geometry.

Execute:
```pseudocode
FUNCTION generate_models(stochastic_grammar, spatial_seed_map, generation_bounds):
    model_instances = []
    generation_queue = [initial_axiom]

    WHILE generation_queue NOT empty:
        current_shape = generation_queue.pop()

        // Rule matching and selection
        applicable_rules = find_applicable_rules(current_shape, stochastic_grammar)
        selected_rule = select_rule_stochastically(applicable_rules, current_shape.position)

        // Rule application
        IF selected_rule.type == "split":
            child_shapes = apply_split_rule(current_shape, selected_rule)
            generation_queue.extend(child_shapes)
        ELIF selected_rule.type == "substitute":
            new_shape = apply_substitute_rule(current_shape, selected_rule)
            generation_queue.append(new_shape)
        ELSE: // terminal rule
            geometry = instantiate_geometry(current_shape, selected_rule)
            model_instances.append(geometry)

    RETURN model_instances
```

**Generation Algorithms:**
- Recursive descent parsing with stochastic selection
- Spatial partitioning for efficient rule lookup: `O(log n)` complexity
- Memory management with geometry instancing
- Level-of-detail generation based on viewing distance

**Optimization Techniques:**
- Frustum culling: `visible = dot(plane_normal, object_center) + plane_distance > -object_radius`
- Occlusion culling: `visible = !occluded_by_previous_objects(object_bounds)`
- Instancing: `render_instances(geometry_id, transform_matrices[])`

**Output Requirements:**
- Complete 3D scene with hierarchical transforms
- Material assignments and UV coordinates
- Collision geometry for interactive applications
- Metadata for post-processing (object IDs, semantic labels)

**Debug Output:** Generation statistics, memory usage, performance profiling, rule application frequency.

---

## Implementation Constraints

### Algorithmic Requirements
- **NO** neural networks, transformers, or learned models
- **USE** deterministic algorithms with controlled randomness
- **IMPLEMENT** classical computer graphics techniques only
- **ENSURE** reproducible results with seed control

### Performance Targets
- Generation time: `O(n log n)` where n is output complexity
- Memory usage: Linear in output size with constant factor < 10
- Real-time capability: > 30 FPS for interactive applications

### Quality Metrics
- Visual coherence: Consistent style across all generated elements
- Structural validity: No geometric intersections or impossible configurations
- Parameter coverage: All input parameters influence final output
- Variation richness: Sufficient diversity without repetition

## Execution Protocol

1. **PARSE** user topic into structured requirements
2. **EXECUTE** each phase sequentially with full debug output
3. **VALIDATE** outputs against quality metrics at each phase
4. **GENERATE** final models with performance profiling
5. **REPORT** complete generation statistics and asset inventory

## Error Handling
- **Constraint Violations:** Automatically adjust parameters to satisfy constraints
- **Infinite Recursion:** Implement maximum recursion depth limits
- **Memory Overflow:** Use streaming generation for large scenes
- **Invalid Geometry:** Apply automatic mesh repair algorithms

## Output

Write all outputs (pseudocode, parameter tables, grammar rules, generation statistics) to `output/{topic-slug}/` where `{topic-slug}` is derived from the topic in lowercase kebab-case. Create the directory and a `.context.json` if needed. Display a summary of the generation pipeline results to the user.
