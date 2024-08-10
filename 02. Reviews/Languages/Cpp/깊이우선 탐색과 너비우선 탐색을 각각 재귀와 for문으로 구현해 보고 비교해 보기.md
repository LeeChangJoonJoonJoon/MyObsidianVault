

2024-01-13

----
#자료구조 #CS #Cpp #DFS #BFS #재귀 #스택오버플로우 #이진트리

## 개요
그룹장님이 나에게 숙제를 주셨다.
두 탐색 방법을 C++ 코드로 구현해 보고 [[시간복잡도, 공간복잡도]]를 비교해 보자.

## 내용
두 탐색 방법은 이진트리와 같은 비선형구조에서 자료를 검색하는 방법이다.
전자인 DFS는 간단히 말하여 재귀함수의 동작과 같이 가장 깊은 곳까지 들어간 뒤 원하는 값이 없으면 바로 이전 분기로 빠져나와 다른 자식에게 들어가 값을 검사하는 것을 말한다.
다음과 같이 만든 언리얼 구조체 내에서 value가 `nullptr`이 나올 때까지 값을 검사하는 게 대표적인 깊이우선 탐색이다.
```cpp
USTRUCT()
struct FParsedData_Fractal
{
	GENERATED_BODY()
	TMap<FString, FParsedData_Fractal*> MapOfJson;
};
```

여기서는 이진트리를 다음과 같이 만들어 보았다.
```cpp
struct NodeStruct  
{  
    int StructNum;  
  
    NodeStruct* pBackNode_L;  
    NodeStruct* pBackNode_R;  
};

void BinTree::MakeNodes(int _NumOfNodes, NodeStruct* _pFrontNode)  
{  
    if (_NumOfNodes > 0)  
    {  
        _NumOfNodes--;  
        auto* pBackNode_1 = new NodeStruct;  
        _pFrontNode->pBackNode_L = pBackNode_1;  
        MakeNodes(_NumOfNodes, pBackNode_1);  
  
//        _NumOfNodes--;  
        auto* pBackNode_2 = new NodeStruct;  
        _pFrontNode->pBackNode_R = pBackNode_2;  
        MakeNodes(_NumOfNodes, pBackNode_2);  
    } else  
    {  
        return;  
    }  
}
```

노드 개수를 조정하는 방식으로 이진트리를 만드는 법도 있다. 
이 때, 선입선출의 큐를 이용하여 편리하게 만들 수 있다.
```cpp
void BinTree::MakeNodes_loop(int _NumOfNodes, NodeStruct* _pNodeSelf)  
{  
    bool bIsLeftNode = true;  
  
    queue<NodeStruct*> QueueOfNodes;  
    QueueOfNodes.push(_pNodeSelf);  
    _NumOfNodes--; // 이미 하나 만들었으니까 1을 빼준다.  
  
    NodeStruct* NodeOfQueue;  
    while (!QueueOfNodes.empty())  
    {        // 우선 while 스코프 내에서만 쓰이는 변수에 담는다.  
        NodeOfQueue = QueueOfNodes.front();  
        // 그리고 담았으니 큐에서 삭제  
        QueueOfNodes.pop();  
  
        /////////////////////////////////////////// Left ///////////////////////////////////////////  
        _NumOfNodes--;  
        if (_NumOfNodes < 0) break;  
  
        auto* pLeftNode = new NodeStruct;  
        NodeOfQueue->pBackNode_L = pLeftNode;  
        pLeftNode->pFrontNode = NodeOfQueue;  
  
        QueueOfNodes.push(pLeftNode);  
        ////////////////////////////////////////////////////////////////////////////////////////////  
  
  
        /////////////////////////////////////////// Right ///////////////////////////////////////////        
        _NumOfNodes--;  
        if (_NumOfNodes < 0) break;  
  
        auto* pRightNode = new NodeStruct;  
        NodeOfQueue->pBackNode_R = pRightNode;  
        pRightNode->pFrontNode = NodeOfQueue;  
  
        QueueOfNodes.push(pRightNode);  
        ////////////////////////////////////////////////////////////////////////////////////////////  
    }  
  
	while (!QueueOfNodes.empty()) QueueOfNodes.pop();  
	NodeOfQueue = nullptr;  
	delete NodeOfQueue;
}
```

이제 이를 하나하나 방문하여 각 노드별로 숫자를 채워넣는 로직을 추가해볼 것임.

### 깊이우선 탐색을 재귀로 구현하기
먼저 깊이우선 탐색을 재귀로 구현해 보았다.
```cpp
void BinTree::DFS_rec(NodeStruct*& _prTree)  
{  
    _prTree->bIsSearched = 1;  
  
    if (_prTree->pBackNode_L && _prTree->pBackNode_L->bIsSearched != 1)  
    {  
        // 왼쪽으로 타고 내려가는 로직  
        DFS_rec(_prTree->pBackNode_L);  
    }  
    else if (_prTree->pBackNode_R && _prTree->pBackNode_R->bIsSearched != 1)  
    {  
        // 오른쪽으로 타고 내려가는 로직  
        DFS_rec(_prTree->pBackNode_R);  
    }  
    else if (_prTree->pFrontNode) // 올라가는 로직  
    {  
        DFS_rec(_prTree->pFrontNode);  
    }  
}
```

디버깅을 통해 모든 노드가 1이 되는 걸 확인했다.
다만 여기서 스택오버플로우 문제가 발생한다.
`BinTree::MakeNodes(int _NumOfNodes, NodeStruct* _pFrontNode)` 함수로 만든 이진트리를 만들 때 `_NumOfNodes` 인수에 10을 넣을 때까진 아무 문제 없이 실행되다가, 17 정도의 숫자를 넣을 때부터 다음과 같은 메시지가 뜨면서 종료된다.
```cpp
/Users/changjoonlee/Documents/Cpp/MyLab/CompareDFSnBFS/cmake-build-debug/CompareDFSnBFS
Type Number : 17

Process finished with exit code 139 (interrupted by signal 11:SIGSEGV)
```

또, 디버깅 세션으로 실행하게 되면 다음과 같이 보여준다.
![[Pasted image 20240121163852.png]]

구글에 검색을 해보니, 이 `exit code 139`는 스택오버플로우 상황일 때 발생하는 메시지였다.
> Your code creates a stack overflow because it recursively deletes the next node from the destructor of the previous one. When you have a million elements this results in a million recursions. Stack size is limited to around 1-8MB on most platforms which even if each function call only used 8 bytes of stack (generally it'd be more) would cause an overflow for a million elements.
> 출처: https://stackoverflow.com/questions/64329877/c-process-finished-with-exit-code-139-interrupted-by-signal-11-sigsegv

번역하면, 해당 답변은 소멸자에서 재귀가 발생했는데, 이 재귀를 실행키 위한 스택 크기가 부족했다는 것이다. 
일반적으로 함수 호출을 위해서는 원래 그 함수가 호출된 지점과 호출되어서 배정된 메모리 지점이 스택에 저장된다.
우리가 디버깅 세션의 왼쪽 창에서 봤던 '호출스택'이란 단어가 이 스택 공간 내의 주소들을 보여주는 것이었다.
이 용량이 제한돼 있기에 발생하는 현상이라는 것이다.
그리고 이 제한 때문에 디버깅 세션에선 `EXC_BAD_ACCESS`가 터졌던 것이다.

### 깊이우선 탐색을 반복문으로 구현하기
다음은 깊이우선 탐색을 반복문으로 실행하는 함수이다.
```cpp
void BinTree::DFS_loop(NodeStruct* _pTree)  
{  
    _pTree->bIsSearched = 1;  
    NodeStruct* pTree_temp = _pTree;  
  
    for (;;)  
    {  
        if (pTree_temp->pBackNode_L && pTree_temp->pBackNode_L->bIsSearched != 1)  
        {  
            // 왼쪽으로 타고 내려가는 로직  
            pTree_temp->pBackNode_L->bIsSearched = 1;  
            pTree_temp = pTree_temp->pBackNode_L;  
        }  
        else if (pTree_temp->pBackNode_R && pTree_temp->pBackNode_R->bIsSearched != 1)  
        {  
            // 오른쪽으로 타고 내려가는 로직  
            pTree_temp->pBackNode_R->bIsSearched = 1;  
            pTree_temp = pTree_temp->pBackNode_R;  
        }  
        else if (pTree_temp->pFrontNode) // 올라가는 로직  
        {  
            pTree_temp = pTree_temp->pFrontNode;  
        }  
        else  
        {  
            break;  
        }  
    }  
}
```

재귀와 반복문의 실행시간을 비교하고자 다음과 같이 메인함수를 구현했다.
```cpp
int main()  
{  
    int num;  
    cout << "Type Number : ";  
    cin >> num;  
  
    BinTree TreeTest;  
    TreeTest.SetBinTree(num);  
  
    //////////////////////////////////////////////////////////////  
    clock_t start_01, finish_01;  
    double duration_01;  
  
    start_01 = clock();  
    NodeStruct* pBinTree_01 = TreeTest.GetBinTree();  
    TreeTest.DFS_rec(pBinTree_01);  
    finish_01 = clock();  
    duration_01 = (double)(finish_01 - start_01);  
  
    //////////////////////////////////////////////////////////////  
    clock_t start_02, finish_02;  
    double duration_02;  
  
    start_02 = clock();  
    NodeStruct* pBinTree_02 = TreeTest.GetBinTree();  
    TreeTest.DFS_loop(pBinTree_02);  
    finish_02 = clock();  
    duration_02 = (double)(finish_02 - start_02);  
  
    cout << "time spent on recursive DFS : " << duration_01 << endl;  
    cout << "time spent on loop DFS : " << duration_02 << endl;  
  
    return 0;  
}
```

입력 값으로 10을 넣으면 다음과 같은 결과가 나온다.
```cpp
/Users/changjoonlee/Documents/Cpp/MyLab/CompareDFSnBFS/cmake-build-debug/CompareDFSnBFS
Type Number : 10
time spent on recursive DFS : 166
time spent on loop DFS : 2

Process finished with exit code 0
```

반복문으로 구현했을 때 검색이 훨씬 빨랐고, 스택 크기도 훨씬 적게 (사실상 1이다) 먹었다.
재귀호출에는 스택 크기만 문제가 되는 게 아닌 것이다.
호출하고 복귀하는 데에는 다시 [[컨텍스트 스위치(Context Switch)]]가 발생하고 이것이 성능 부하로 이어진다.

### 너비우선 탐색을 재귀로 구현하기
큐의 특성인 선입선출을 이용하여 간단하게 구현해 볼 수 있다.
```cpp
void BinTree::BFS_rec(queue<NodeStruct*>& _rVisitedQueue)  
{  
    if (_rVisitedQueue.empty()) return;  
  
    auto pNodeOfQueue = _rVisitedQueue.front();  
    _rVisitedQueue.pop();  
  
    pNodeOfQueue->bIsVisited = 1;  
    if (pNodeOfQueue->pBackNode_L) _rVisitedQueue.push(pNodeOfQueue->pBackNode_L);  
    if (pNodeOfQueue->pBackNode_R) _rVisitedQueue.push(pNodeOfQueue->pBackNode_R);  
  
    pNodeOfQueue = nullptr;  
    delete pNodeOfQueue;  
  
    BFS_rec(_rVisitedQueue);  
}
```

재귀적인 과정으로 트리를 탐색하긴 하지만 호출의 순서는 일반적으로 반복문으로 구현했을 때와 별반 다르지 않아 싱겁긴 하지만... 어쨌든 재귀를 쓰긴 했다.

### 너비우선 탐색을 루프로 구현하기
위의 방법과 동일하다.
```cpp
void BinTree::BFS_loop(NodeStruct* _pTree)  
{  
    queue<NodeStruct*> QueueOfNodes;  
    QueueOfNodes.push(_pTree);  
  
    NodeStruct* pNode_temp;  
    while (!QueueOfNodes.empty())  
    {  
        pNode_temp = QueueOfNodes.front();  
        QueueOfNodes.pop();  
  
        pNode_temp->bIsVisited = 1;  
        if (pNode_temp->pBackNode_L) QueueOfNodes.push(pNode_temp->pBackNode_L);  
        if (pNode_temp->pBackNode_R) QueueOfNodes.push(pNode_temp->pBackNode_R);  
    }  
  
    pNode_temp = nullptr;  
    delete pNode_temp;  
    while (!QueueOfNodes.empty()) QueueOfNodes.pop();  
}
```

