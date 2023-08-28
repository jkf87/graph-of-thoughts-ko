# Graph of Thoughts (GoT)

<p align="center">
  <img src="paper/pics/preview.svg">
</p>

이것은 [생각의 그래프]의 공식 구현입니다: 대규모 언어 모델로 정교한 문제 해결하기](https://arxiv.org/pdf/2308.09687.pdf). 
이 프레임워크는 대규모 언어 모델(LLM)을 엔진으로 하여 자동으로 실행되는 작업 그래프(GoO)로 모델링하여 복잡한 문제를 해결할 수 있는 기능을 제공합니다. 
이 프레임워크는 유연하고 확장 가능하도록 설계되어 새로운 GoT 접근 방식을 사용하여 문제를 해결할 수 있을 뿐만 아니라 CoT 또는 ToT와 같은 이전 접근 방식과 유사한 GoO를 구현할 수도 있습니다.

## Setup Guide

이 프레임워크를 사용하려면 Python 3.8 이상이 정상적으로 설치되어 있어야 합니다.

### Installing GoT

다음 두 가지 설치 방법 중 하나를 실행하기 전에 Python 환경(있는 경우)을 미리 활성화해야 합니다. `graph_of_thoughts`만 사용하려는 사용자라면 PyPI에서 직접 설치할 수 있습니다:
```bash
pip install graph_of_thoughts
```
개발자이고 코드를 수정하려는 경우 소스에서 편집 가능한 모드로 설치할 수 있습니다:
```bash
git clone https://github.com/spcl/graph-of-thoughts.git
cd graph-of-thoughts
pip install -e .
```

### Configuring the LLM

프레임워크를 사용하려면 LLM에 액세스할 수 있어야 합니다.[Controller README](graph_of_thoughts/controller/README.md)를 사용하여 원하는 LLM을 구성할 수 있습니다.

## Quick Start

다음 코드 스니펫은 프레임워크를 사용하여 CoT와 유사한 접근 방식을 사용하여 32개의 숫자 목록에 대한 정렬 문제를 해결하는 방법을 보여줍니다. 코드를 실행하기 전에 [설정 가이드(#setup-가이드) 코드를 실행하기 전에.

```python
from examples.sorting.sorting_032 import SortingPrompter, SortingParser, utils
from graph_of_thoughts import controller, operations

# Problem input

to_be_sorted = "[0, 2, 6, 3, 8, 7, 1, 1, 6, 7, 7, 7, 7, 9, 3, 0, 1, 7, 9, 1, 3, 5, 1, 3, 6, 4, 5, 4, 7, 3, 5, 7]"

# Create the Graph of Operations
gop = operations.GraphOfOperations()
gop.append_operation(operations.Generate())
gop.append_operation(operations.Score(scoring_function=utils.num_errors))
gop.append_operation(operations.GroundTruth(utils.test_sorting))

# Configure the Language Model (Assumes config.json is in the current directory with OpenAI API key)
lm = controller.ChatGPT("config.json", model_name="chatgpt")

# Create the Controller
ctrl = controller.Controller(
  lm, 
  gop, 
  SortingPrompter(), 
  SortingParser(),
  # The following dictionary is used to configure the initial thought state
  {
    "original": to_be_sorted,
    "current": "",
    "method": "cot"
  }
)

# Run the Controller and generate the output graph
ctrl.run()
ctrl.output_graph("output_cot.json")
```

보다 정교한 GoT 접근 방식을 실행하려면 다음 코드 스니펫을 사용할 수 있습니다.

```python
from examples.sorting.sorting_032 import SortingPrompter, SortingParser, got, utils
from graph_of_thoughts import controller, operations

# Problem input

to_be_sorted = "[0, 2, 6, 3, 8, 7, 1, 1, 6, 7, 7, 7, 7, 9, 3, 0, 1, 7, 9, 1, 3, 5, 1, 3, 6, 4, 5, 4, 7, 3, 5, 7]"

# Retrieve the Graph of Operations
gop = got()

# Configure the Language Model (Assumes config.json is in the current directory with OpenAI API key)
lm = controller.ChatGPT("config.json", model_name="chatgpt")

# Create the Controller
ctrl = controller.Controller(
  lm, 
  gop, 
  SortingPrompter(), 
  SortingParser(),
  # The following dictionary is used to configure the initial thought state
  {
    "original": to_be_sorted,
    "current": "",
    "phase": 0,
    "method": "got"
  }
)

# Run the Controller and generate the output graph
ctrl.run()
ctrl.output_graph("output_got.json")
```
You can compare the two results by inspecting the output graphs `output_cot.json` and `output_got.json`.  
The final thought states' scores indicate the number of errors in the sorted list.

출력 그래프 `output_cot.json`와 `output_got.json을 검사하여 두 결과를 비교할 수 있습니다. 
최종 생각 상태의 점수는 정렬된 목록에서 오류의 수를 나타냅니다.

## Documentation
The paper gives a high-level overview of the framework and its components.  
In order to understand the framework in more detail, you can read the documentation of the individual modules.  
Especially the [Controller](graph_of_thoughts/controller/README.md) and [Operations](graph_of_thoughts/operations/README.md) modules are important for understanding how to make the most out of the framework.  
We took extra care to fully document the code, so that you can easily understand how it works and how to extend it.

이 백서에서는 프레임워크와 그 구성 요소에 대한 개괄적인 개요를 제공합니다. 
프레임워크를 더 자세히 이해하려면 개별 모듈의 설명서를 읽어보세요. 
특히[Controller](graph_of_thoughts/controller/README.md) 및 [Operations](graph_of_thoughts/operations/README.md) 모듈은 프레임워크를 최대한 활용하는 방법을 이해하는 데 중요합니다. 
코드의 작동 방식과 확장 방법을 쉽게 이해할 수 있도록 코드를 완전히 문서화하기 위해 각별히 신경을 썼습니다.

## Examples

The [examples](examples) directory contains several examples of problems that can be solved using the framework, including the ones presented in the paper.  
It is a great starting point for learning how to use the framework to solve real problems.  
Each example contains a `README.md` file with instructions on how to run it and play with it. The code is fully documented and should be easy to follow.

[examples](examples) 디렉터리에는 프레임워크를 사용하여 해결할 수 있는 몇 가지 문제 예제가 포함되어 있습니다, 
이 문서에 제시된 문제를 포함하여. 프레임워크를 사용하여 실제 문제를 해결하는 방법을 배우기 위한 훌륭한 출발점입니다. 
각 예제에는 실행 및 플레이 방법에 대한 지침이 포함된 `README.md` 파일이 포함되어 있습니다. 
코드는 완전히 문서화되어 있으며 쉽게 따라할 수 있습니다.

## Paper Results

You can run the experiments from the paper by following the instructions in the [examples](examples) directory.  
However, if you just want to inspect and replot the results, you can use the [paper](paper) directory.

[examples](examples) 이 디렉터리에 있는 지침에 따라 이 문서에서 실험을 실행할 수 있습니다.
그러나 결과를 검사하고 다시 플롯하려는 경우, [paper](paper) 디렉터리를 사용하세요.

## Citations

If you find this repository valuable, please give it a star!  
Got any questions or feedback? Feel free to reach out to [nils.blach@inf.ethz.ch](mailto:nils.blach@inf.ethz.ch) or open an issue.  
Using this in your work? Please reference us using the provided citation:

이 리포지토리가 유용하다고 생각되면 별표를 달아주세요! 
질문이나 피드백이 있으신가요? 
[nils.blach@inf.ethz.ch](mailto:nils.blach@inf.ethz.ch)로 언제든지 문의해 주세요. 또는 깃헙 이슈를 열 수 있습니다. 
이 자료를 업무에 사용하시나요? 제공된 인용문을 사용하여 참조해 주세요.

```bibtex
@misc{besta2023got,
  title = {{Graph of Thoughts: Solving Elaborate Problems with Large Language Models}},
  author = {Besta, Maciej and Blach, Nils and Kubicek, Ales and Gerstenberger, Robert and Gianinazzi, Lukas and Gajda, Joanna and Lehmann, Tomasz and Podstawski, Micha{\l} and Niewiadomski, Hubert and Nyczyk, Piotr and Hoefler, Torsten},
  year = 2023,
  eprinttype = {arXiv},
  eprint = {2308.09687}
}
```
