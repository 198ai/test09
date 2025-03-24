@startuml
participant "キーワード検索フラグメント" as Fragment
participant "リポジトリ" as Repository
participant "Flutter" as Flutter
participant "APIサーバー" as API

activate Fragment

Fragment -> Repository: onLoadMore() 呼び出し
activate Repository

Repository -> Flutter: onScrolledBottom() 呼び出し
activate Flutter

Flutter -> API: /search APIリクエスト
activate API

API --> Flutter: 検索結果items\n& ヒット数 返却
deactivate API

Flutter --> Repository: 検索結果 通知
deactivate Flutter

Repository -> Repository: resultContents更新\nUI状態更新
Repository --> Fragment: 更新完了コールバック
deactivate Repository

activate Fragment
Fragment -> Fragment: updateUI() 実行

alt UI状態 == UPDATE の場合
    alt resultContentsが空の場合
        Fragment -> Fragment: 状態のみ更新
    else
        Fragment -> Fragment: 新規リストに追加\nUI更新
    end
end
deactivate Fragment
@enduml
