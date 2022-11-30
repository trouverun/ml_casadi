[![ML-CasADi CI](https://github.com/TUM-AAS/ml-casadi/actions/workflows/python-test.yml/badge.svg)](https://github.com/TUM-AAS/ml-casadi/actions/workflows/python-test.yml)

# ML-CasADi
This is the underlying framework enabling Neural-MPC in our paper

`Neural-MPC: Deep Learning Model Predictive Control for Quadrotors and Agile Robotic Platforms`

[Arxiv Link](https://arxiv.org/pdf/2203.07747)

This framework enables PyTorch Models to be used as CasADi functions and subsequently in Acados optimal control problems.

## Examples
### GPytorch gaussian process model as first- or second order approximation
```
import ml_casadi.torch as mc
import casadi as ca
import numpy as np
import torch

inputs = torch.stack([torch.linspace(-1, 1, 100), torch.linspace(-1, 1, 100)], dim=1)inputs = torch.as_tensor(inputs)
outputs = torch.stack([5*torch.sin(x[:, 0]), 5*torch.cos(x[:, 1])], dim=1)

# Model is trained when instantiated:
gp_model = mc.nn.GPModel(inputs, targets, input_size=2, output_size=2, device="cuda:0")

casadi_quad_approx_sym_out = gp_model.approx(casadi_sym_inp, order=2)
casadi_quad_approx_func = cs.Function('gp_quad',
                                      [casadi_sym_inp,
                                       gp_model.sym_approx_params(flat=True, order=2)],
                                      [casadi_quad_approx_sym_out])
casadi_quad_approx_param = gp_model.approx_params(flat=True, order=2, a=np.zeros((1, 2)))

test_input = np.zeros(2)
casadi_out = casadi_quad_approx_func(test_input, casadi_quad_approx_param).toarray()
casadi_mu = casadi_out[0, :] # [output_1_mu, output_2_mu]
casadi_std = casadi_out[1, :] # [output_1_std, output_2_std]

```
### Arbitrary PyTorch Model as first- or second order approximation
```
size_in = 6
size_out = 3
model = mc.TorchMLCasadiModuleWrapper(
    torch_module,
    input_size=size_in,
    output_size=size_out)
casadi_sym_inp = ca.MX.sym('inp',size_in)
casadi_sym_out = model.approx(casadi_sym_inp, order=1)
casadi_func = ca.Function('model_approx_wrapper',
                          [casadi_sym_inp, model.sym_approx_params(order=1, flat=True)],
                          [casadi_sym_out])

inp = np.ones([1, size_in])  # torch needs batch dimension
casadi_param = model.approx_params(inp, order=1, flat=True)  # order=2
casadi_out = casadi_func(inp.transpose(-2, -1), casadi_param)   # transpose for vector rep. expected by casadi

t_out = model(torch.tensor(inp, dtype=torch.float32))

print(casadi_out)
print(t_out)
```

Using Acados, the approximation parameters can be passed to the optimal control problem via `acados_ocp_solver.set(n, 'p', casadi_param)`.

### Specific PyTorch Model without approximation
```
import ml_casadi.torch as mc
import casadi as cs

model = mc.MultiLayerPerceptron(
    input_size=size_in,
    hidden_size=hidden_size,
    output_size=size_out,
    n_hidden=n_hidden,
    activation='relu')
    
casadi_sym_inp = cs.MX.sym('inp', size_in)
casadi_sym_out = model(casadi_sym_inp)

casadi_func = cs.Function('model_approx_wrapper',
                          [casadi_sym_inp],
                          [casadi_sym_out])

casadi_out = casadi_func(input, casadi_param)
```

## Citing
If you use our work please cite our paper
```
@article{salzmann2022neural,
  title={Neural-MPC: Deep Learning Model Predictive Control for Quadrotors and Agile Robotic Platforms},
  author={Salzmann, Tim and Kaufmann, Elia and Pavone, Marco and Scaramuzza, Davide and Ryll, Markus},
  journal={arXiv preprint arXiv:2203.07747},
  year={2022}
}
```