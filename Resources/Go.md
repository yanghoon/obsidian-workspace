
## Templates

### Go 템플릿 연습

* [Go Template Preview](https://gotemplate.io/)
* [Go 템플릿 완벽 가이드 - 문법과 내부 동작](https://mycodings.fly.dev/blog/2024-05-18-go-templates-complete-guide)

### Go 템플릿 예제

```go
{{/* if문 사용 예제 */}}
{{if .Name}}
  Hello, {{.Name}}!
{{else if .Guest}}
  Hello, Guest!
{{else}}
  Hello, Anonymous!
{{end}}

{{/* map 반복문 예제 */}}
{{range $key, $value := .UserInfo}}
  {{$key}}: {{$value}}
{{end}}

{{/* list 반복문 예제 */}}
{{range .Tasks}}
  - {{.}}
{{end}}
```

- `.Name`, `.Guest`는 조건 검사 변수입니다. nil, 0, 빈 값은 false 처리됩니다.
- map 반복문은 `range $key, $value := map` 형태로 키와 값을 순회합니다.
- list 반복문은 `range .List` 형태로 요소를 순회합니다.

예시 데이터 구조:

```go
type PageData struct {
  Name     string
  Guest    bool
  UserInfo map[string]string
  Tasks    []string
}
```

이 템플릿은 필드에 따라 조건을 분기하고, map과 리스트를 각각 반복해 출력합니다