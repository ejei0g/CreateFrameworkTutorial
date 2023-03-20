### Ref
https://www.kodeco.com/17753301-creating-a-framework-for-ios

### Why
동일한 코드가 서로 다른 앱에서 사용될 때
내가 만든 코드를 다른 개발자와 공유하고 싶을 때
iOS SDK처럼 기능별로 API를 분리하고 싶을 때
다른 3rd Party 라이브러리처럼 배포하고 싶을 때

### What
- 새로운 프레임워크 생성하기
- 기존 코드 migration하기
- 앱에 다시 import하기
- XCFramework (binary framework) 빌드하기
- Pack it as an uber-portable Swift Package
- Set up repository for your Swift Package and publish it

### **기존에 생성된 프레임워크를  CocoaPod를 사용해서 라이브러리로 생성하고 릴리즈하기**

1. Git Repo에 등록 및 `tag` 추가 
    
    <aside>
    💡 tag 기준으로 가져오기 때문에 추가하기
    
    </aside>
    
    ```jsx
    git tag 0.0.1
    git push --tags
    ```
    
2. `.podspec` 파일 생성
    
    [CocoaPods.org](https://guides.cocoapods.org/making/specs-and-specs-repo.html)
    
    ```jsx
    pod spec create {name}
    ```
    
3. `.podspec` 파일 설정하기
    
    [CocoaPods.org](https://guides.cocoapods.org/syntax/podspec.html#license)
    
    ```jsx
    // 🚨 주의를 요함! 주이가 필요한 옵션들
    
    spec.version      = "0.0.3" // 라이브러리 업데이트시 해당 파라미터를 업데이트
    spec.platform     = :ios, "14.0" // 최소 지원보다 낮은 프로젝트에서 pod install 불가
    
    // git주소와 태그 설정 (위에서 설정한 태그정보로 가져옴)
    spec.source       = { :git => "https://github.com/ejei0g/CreateFrameworkTutorial.git",
    	                    :tag => "#{spec.version}" }
    
    // 라이브러리에서 필요한 파일 위치를 명시 (swift 파일을 위해 .swift 붙여주기)
    spec.source_files  = "2-Framework/CalendarControl", "2-Framework/CalendarControl/**/*.{h,m,swift}"
    
    // swift version 명시 (아래 Error handle 참고)
    spec.swift_versions = ["4.2", "5.0"]
    
    // ARM 관련 옵션인데 아직 해당 옵션이 필요한 에러는 경험해보지 못함(m1 이슈같긴한데, 시뮬을 m1으로 바꿔주는 역할인듯?)
    #  spec.pod_target_xcconfig = { 'EXCLUDED_ARCHS[sdk=iphonesimulator*]' => 'arm64'}
    #  spec.user_target_xcconfig = { 'EXCLUDED_ARCHS[sdk=iphonesimulator*]' => 'arm64'}
    ```
    
4. License 생성 (source의 tag기준으로 LICENSE파일을 찾을 수 있어야한다.)
    
    <aside>
    💡 Git에서 루트에 있어야 인식하나봄 (폴더 내부에 있을 경우, 디렉토리를 명시해줘도 못찾는듯?)
    
    </aside>
    
5. `.podspec` 정상적으로 동작하는지 다음명령으로 테스트
    
    ```jsx
    pod spec lint --allow-warnings // warning이 있어도 컴파일 가능한 옵션
    ```
    
6. 배포
    1. Private
        
        [CocoaPods.org](https://guides.cocoapods.org/making/private-cocoapods)
        
        <aside>
        💡 테스트는 하나의 레포에서 소스와 spec을 관리했는데, 분리시키는게 좀 더 좋을 거 같네요
        
        </aside>
        
        ```jsx
        // 개인 레포 등록 (내부에서 사용할 경우, 접근 권한만 풀어주면 됨)
        pod repo add {name} {source_url} // git 주소 or local url (private 주소 가능)
        
        // repo test
        $ cd ~/.cocoapods/repos/REPO_NAME
        $ pod repo lint .
        
        // 변경사항 푸쉬
        pod repo push {spec_repo_name} {podspec_filename.podspec} 
        ```
        
        ```jsx
        // 일반적인 Release 흐름
        
        1. 라이브러리 변경 작업 (버그 수정, 기능 추가 등)
        2. 변경 내용 tag와 함께 릴리즈 (tag 1.0.1)
        3. .podspec 편집 (라이브러리 버전 변경 1.0.0 => 1.0.1)
        4. pod {trunk or repo} push {버전이 변경된 podspec 파일}.podspec
        ```
        
    2. Public
        
        [CocoaPods.org](https://guides.cocoapods.org/making/getting-setup-with-trunk)
        
        `pod trunk` 를 사용해서 push
        
        ```jsx
        pod trunk push {name}.podspec
        ```
        
    3. (참고)  `Bitmovin`에서 cocoapod-specs repo를 관리하는 방식
        
        https://github.com/bitmovin/cocoapod-specs
        

### Error Handle

```jsx
[!] Unable to find the `Ca` repo. If it has not yet been cloned, add it via `pod repo add`.
```

- 개인 레포 등록 없이 푸쉬하는 경우

```jsx
- WARN  | [iOS] swift: The validator used Swift `4.0` by default because no Swift version was specified. To specify a Swift version during validation, add the `swift_versions` attribute in your podspec. Note that usage of a `.swift-version` file is now deprecated.
 CalendarControl.podspec+                                                                                                                                                                                                                                                                buffers
- ERROR | [iOS] xcodebuild: Returned an unsuccessful exit code.
- NOTE  | xcodebuild:  note: Using codesigning identity override: -
- NOTE  | [iOS] xcodebuild:  note: Building targets in dependency order
- ERROR | xcodebuild:  CalendarControl/2-Framework/CalendarControl/CalendarControl/View/CalendarDateCollectionViewCell.swift:44:49: error: type 'UIAccessibilityTraits' (aka 'UInt64') has no member 'button'
- ERROR | xcodebuild:  CalendarControl/2-Framework/CalendarControl/CalendarControl/View/CalendarDateCollectionViewCell.swift:112:25: error: value of type 'UIAccessibilityTraits' (aka 'UInt64') has no member 'insert'
- ERROR | xcodebuild:  CalendarControl/2-Framework/CalendarControl/CalendarControl/View/CalendarDateCollectionViewCell.swift:112:54: error: type 'UIAccessibilityTraits' (aka 'UInt64') has no member 'selected'
- ERROR | xcodebuild:  CalendarControl/2-Framework/CalendarControl/CalendarControl/View/CalendarDateCollectionViewCell.swift:121:25: error: value of type 'UIAccessibilityTraits' (aka 'UInt64') has no member 'remove'
- ERROR | xcodebuild:  CalendarControl/2-Framework/CalendarControl/CalendarControl/View/CalendarDateCollectionViewCell.swift:121:54: error: type 'UIAccessibilityTraits' (aka 'UInt64') has no member 'selected'
- ERROR | xcodebuild:  CalendarControl/2-Framework/CalendarControl/CalendarControl/View/CalendarPickerHeaderView.swift:10:55: error: type 'UIAccessibilityTraits' (aka 'UInt64') has no member 'header'
```

```jsx
// {Name}.podspec
spec.swift_versions = ["5.0"] or ["4.2", "5.0"]
```

왜인지는 모르겠으나, swift 버전을 특정했을 때, 에러 해결

### &ref

[CocoaPods.org](https://guides.cocoapods.org/making/making-a-cocoapod.html)

Tutorial |  [https://www.kodeco.com/5823-how-to-create-a-cocoapod-in-swift#toc-anchor-005](https://www.kodeco.com/5823-how-to-create-a-cocoapod-in-swift#toc-anchor-005)
