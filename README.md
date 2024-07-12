# UCS Alg Node wrap demo

This is a simple demo of how ucs_alg_node is integrated to wrap a specific algorithm


## Installs
To install ucs_alg_node, run the following command:
```bash
pip3 install --force-reinstall ./utils/ucs_alg_node-${version}-py3-none-any.whl
```

## N.B.

The algorithm is wrapped inside a single process, if multiple process is created inside the algorithm, should be killed manually.