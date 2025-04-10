@startuml
class B {

onCreateView()
}

class Fragment1 extends B {

onCreateView() 【Overwrite】
}

class Fragment2 extends B {

onCreateView() 【Overwrite】
}

note right of Fragment2::onCreateView
ロード　fragment2_new.xml
end note

note right of Fragment1::onCreateView
ロード　fragment1_new.xml
end note

class layout << (S,#FF7700) >> {
新增7个XML布局
fragment1_new.xml
fragment2_new.xml
...
}



Fragment1 ..> layout : 使用
Fragment2 ..> layout : 使用
@enduml
