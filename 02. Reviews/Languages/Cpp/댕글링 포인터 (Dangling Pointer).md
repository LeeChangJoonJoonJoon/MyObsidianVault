

2024-02-03

----
#unique_ptr #shared_ptr #Cpp #make_unique #CS

## 개요
[[깊이우선 탐색과 너비우선 탐색을 각각 재귀와 for문으로 구현해 보고 비교해 보기]]에서 이진트리를 만들던 중...
```cpp
void BinTree::MakeNodes_loop(int _NumOfNodes, NodeStruct* _pNodeSelf)  
{  
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

마지막 부분의 `NodeOfQueue = nullptr;`를 쓰는지 여부에 따라 댕글링 포인터 문제가 생겼다 안생겼다 하는 문제를 발견했다.
`NodeOfQueue = nullptr;`를 쓰면, 나중에 저 이진트리를 반복문으로 탐색하는 함수에서 `BAD_ACCESS`문제가 생기지 않았다.
하지만 저걸 쓰지 않은 채 메모리 해제만 해주면 `BAD_ACCESS`가 뜬다.

## 내용
디버깅 세션에 들어가서, `NodeOfQueue = nullptr;`가 있는 행에 중단점을 걸고 히트시켜봤다.
![[Pasted image 20240203193330.png]]

즉, 실재 어떤 객체의 포인터를 가리키고 있는 상태였다.
따라서 내가 `delete`를 쓴 것은 `NodeOfQueue`라는 변수가 차지하는 메모리를 해제한 것이 아니라, 저 변수가 가리키고 있는 어떤 특정 객체의 메모리 위치를 해제한 것이었다.
따라서 `nullptr`을 넣으면 에러가 나지 않았던 것.

다만 `nullptr`을 넣어도 `nullptr`을 대입한 포인터의 메모리 주소는 그대로 남아 있으니 메모리 해제를 잊지 말자.
![[Pasted image 20240203203829.png]]