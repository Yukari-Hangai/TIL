# Bookersのテストコードを読み解く
~~~ruby
1	# frozen_string_literal: true
2	 
3	require 'rails_helper'
4	 
5	RSpec.describe Book, "モデルに関するテスト", type: :model do
6	  describe '実際に保存してみる' do
7	    it "有効な投稿内容の場合は保存されるか" do
8	      expect(FactoryBot.build(:book)).to be_valid
9	    end
10	  end
11	  context "空白のバリデーションチェック" do
12	    it "titleが空白の場合にバリデーションチェックされ空白のエラーメッセージが返ってきているか" do
13	      book = Book.new(title: '', body:'hoge')
14	      expect(book).to be_invalid
15	      expect(book.errors[:title]).to include("can't be blank")
16	    end
17	    it "bodyが空白の場合にバリデーションチェックされ空白のエラーメッセージが返ってきているか" do
18	      book = Book.new(title: 'hoge', body:'')
19	      expect(book).to be_invalid
20	      expect(book.errors[:body]).to include("can't be blank")
21	    end
22	  end
23	end
~~~

**1行目**:# frozen_string_literal: true

文字列リテラルをfreeze（破壊的変更を防止）させています。
RSpecとは関係ありません。記述がなくても動作します。


**2行目**:require 'rails_helper'

これはspec/rails_helper.rbを読み込んでいます。設定などを行うファイルです。


**8行目**:expect(FactoryBot.build(:book)).to be_valid

FactoryBot.build(:book)で作成したBookモデルのインスタンスをexpectに渡して、be_validで有効かを判定しています。


**20行目**:expect(book.errors[:body]).to include("can't be blank")



~~~ruby
1	require 'rails_helper'
2	 
3	describe '投稿のテスト' do
4	  let!(:book) { create(:book,title:'hoge',body:'body') }
5	  describe 'トップ画面(root_path)のテスト' do
6	    before do 
7	      visit root_path
8	    end
9	    context '表示の確認' do
10	      it 'トップ画面(root_path)に一覧ページへのリンクが表示されているか' do
11	        expect(page).to have_link "", href: books_path
12	      end
13	      it 'root_pathが"/"であるか' do
14	        expect(current_path).to eq('/')
15	      end
16	    end
17	  end
18	  describe "一覧画面のテスト" do
19	    before do
20	      visit books_path
21	    end
22	    context '一覧の表示とリンクの確認' do
23	      it "bookの一覧表示(tableタグ)と投稿フォームが同一画面に表示されているか" do
24	        expect(page).to have_selector 'table'
25	        expect(page).to have_field 'book[title]'
26	        expect(page).to have_field 'book[body]'
27	      end
28	      it "bookのタイトルと感想を表示し、詳細・編集・削除のリンクが表示されているか" do
29	          (1..5).each do |i|
30	            Book.create(title:'hoge'+i.to_s,body:'body'+i.to_s)
31	          end
32	          visit books_path
33	          Book.all.each_with_index do |book,i|
34	            j = i * 3
35	            expect(page).to have_content book.title
36	            expect(page).to have_content book.body
37	            # Showリンク
38	            show_link = find_all('a')[j]
39	            expect(show_link.native.inner_text).to match(/show/i)
40	            expect(show_link[:href]).to eq book_path(book)
41	            # Editリンク
42	            show_link = find_all('a')[j+1]
43	            expect(show_link.native.inner_text).to match(/edit/i)
44	            expect(show_link[:href]).to eq edit_book_path(book)
45	            # Destroyリンク
46	            show_link = find_all('a')[j+2]
47	            expect(show_link.native.inner_text).to match(/destroy/i)
48	            expect(show_link[:href]).to eq book_path(book)
49	          end
50	      end
51	      it 'Create Bookボタンが表示される' do
52	        expect(page).to have_button 'Create Book'
53	      end
54	    end
55	    context '投稿処理に関するテスト' do
56	      it '投稿に成功しサクセスメッセージが表示されるか' do
57	        fill_in 'book[title]', with: Faker::Lorem.characters(number:5)
58	        fill_in 'book[body]', with: Faker::Lorem.characters(number:20)
59	        click_button 'Create Book'
60	        expect(page).to have_content 'successfully'
61	      end
62	      it '投稿に失敗する' do
63	        click_button 'Create Book'
64	        expect(page).to have_content 'error'
65	        expect(current_path).to eq('/books')
66	      end
67	      it '投稿後のリダイレクト先は正しいか' do
68	        fill_in 'book[title]', with: Faker::Lorem.characters(number:5)
69	        fill_in 'book[body]', with: Faker::Lorem.characters(number:20)
70	        click_button 'Create Book'
71	        expect(page).to have_current_path book_path(Book.last)
72	      end
73	    end
74	    context 'book削除のテスト' do
75	      it 'bookの削除' do
76	        expect{ book.destroy }.to change{ Book.count }.by(-1)
77	        # ※本来はダイアログのテストまで行うがココではデータが削除されることだけをテスト
78	      end
79	    end
80	  end
81	  describe '詳細画面のテスト' do
82	    before do
83	      visit book_path(book)
84	    end
85	    context '表示の確認' do
86	      it '本のタイトルと感想が画面に表示されていること' do
87	        expect(page).to have_content book.title
88	        expect(page).to have_content book.body
89	      end
90	      it 'Editリンクが表示される' do
91	        edit_link = find_all('a')[0]
92	        expect(edit_link.native.inner_text).to match(/edit/i)
93				end
94	      it 'Backリンクが表示される' do
95	        back_link = find_all('a')[1]
96	        expect(back_link.native.inner_text).to match(/back/i)
97				end  
98	    end
99	    context 'リンクの遷移先の確認' do
100	      it 'Editの遷移先は編集画面か' do
101	        edit_link = find_all('a')[0]
102	        edit_link.click
103	        expect(current_path).to eq('/books/' + book.id.to_s + '/edit')
104	      end
105	      it 'Backの遷移先は一覧画面か' do
106	        back_link = find_all('a')[1]
107	        back_link.click
108	        expect(page).to have_current_path books_path
109	      end
110	    end
111	  end
112	  describe '編集画面のテスト' do
113	    before do
114	      visit edit_book_path(book)
115	    end
116	    context '表示の確認' do
117	      it '編集前のタイトルと感想がフォームに表示(セット)されている' do
118	        expect(page).to have_field 'book[title]', with: book.title
119	        expect(page).to have_field 'book[body]', with: book.body
120	      end
121	      it 'Update Bookボタンが表示される' do
122	        expect(page).to have_button 'Update Book'
123	      end
124	      it 'Showリンクが表示される' do
125	        show_link = find_all('a')[0]
126	        expect(show_link.native.inner_text).to match(/show/i)
127				end  
128	      it 'Backリンクが表示される' do
129	        back_link = find_all('a')[1]
130	        expect(back_link.native.inner_text).to match(/back/i)
131				end  
132	    end
133	    context 'リンクの遷移先の確認' do
134	      it 'Showの遷移先は詳細画面か' do
135	        show_link = find_all('a')[0]
136	        show_link.click
137	        expect(current_path).to eq('/books/' + book.id.to_s)
138	      end
139	      it 'Backの遷移先は一覧画面か' do
140	        back_link = find_all('a')[1]
141	        back_link.click
142	        expect(page).to have_current_path books_path
143	      end
144	    end
145	    context '更新処理に関するテスト' do
146	      it '更新に成功しサクセスメッセージが表示されるか' do
147	        fill_in 'book[title]', with: Faker::Lorem.characters(number:5)
148	        fill_in 'book[body]', with: Faker::Lorem.characters(number:20)
149	        click_button 'Update Book'
150	        expect(page).to have_content 'successfully'
151	      end
152	      it '更新に失敗しエラーメッセージが表示されるか' do
153	        fill_in 'book[title]', with: ""
154	        fill_in 'book[body]', with: ""
155	        click_button 'Update Book'
156	        expect(page).to have_content 'error'
157	      end
158	      it '更新後のリダイレクト先は正しいか' do
159	        fill_in 'book[title]', with: Faker::Lorem.characters(number:5)
160	        fill_in 'book[body]', with: Faker::Lorem.characters(number:20)
161	        click_button 'Update Book'
162	        expect(page).to have_current_path book_path(book)
163	      end
164	    end
165	  end
166	end
~~~

**4行目**:let!(:book) { create(:book,title:'hoge',body:'body') }
データの作成を行い「book」として使用することが可能です。
letで指定されたものは遅延評価といってitの中でbookが出てきたときに始めて実行されます。
今回のlet!は事前評価といいます。beforeと似たような使われ方をされますが、beforeではitブロックの実行時に都度実行されます。
そのためit内で使用しない変数の場合にもbeforeは実行されてしまいます。

**6~8行目**:
~~~ruby
before do 
  visit root_path
end
~~~
visitは指定したパスへの画面遷移を行います。
今回の場合ルートパス（トップ画面）への遷移をbeforeフックとして実行させています。
この理由は、以降の続くit、2つのブロックでルートパス（トップ画面）への遷移を前提条件としているためです。

**11行目**:expect(page).to have_link "", href: books_path
expect(page)で現在visitメソッドなどで開いているページ全体について調べています。
have_linkではそのページの中のaタグ（link_toヘルパー）で、指定した文字列とhref属性を持ったものが存在するかを判定しています。
今回の場合はhrefにbooksの一覧画面への遷移URLが存在するかをチェックしています。

**14行目**:expect(current_path).to eq('/')
expect(current_path)で現在のページURLを調べています。
eqは、指定した値とexpectの値が同値であるかを判定します。

**24行目**:expect(page).to have_selector 'table'
have_selectorでは特定のタグが存在しているか、また特定のタグに特定の文字列があるかなどを判定します。
今回の場合はtableタグが存在しているかを判定しています。
※特定のタグに特定の文字列があるかなどを判定する場合は以下のように記述します。
~~~ruby
expect(page).to have_selector 'p', text: 'テスト'
~~~
この場合、ページの中に「テスト」という文字列があるpタグが存在するか判定します。

**25行目**:expect(page).to have_field 'book[title]'
have_fieldでは指定した値のフォームが存在するか判定します。
今回の場合はbook[title]というname属性のフォームが存在するかで判定しています。
book[title]というname属性は、Bookモデルのtitleカラムを入力するフォームをform_withで作成した際に自動で生成されるものです。
※編集などで指定のフォームに値が表示されているかを判定する場合は以下のように記述します。
~~~ruby
expect(page).to have_field 'book[title]', with: 'テスト'
~~~

**38行目**:show_link = find_all('a')[j]
find_allでは指定した値（タグ）をページ中から全て検索します。
今回の場合はj番目のaタグをshow_linkに代入しています。

**39行目**:expect(show_link.native.inner_text).to match(/show/i)
matchは正規表現を用いて文字列をチェックします。
「/show/i」が正規表現で、Rubyでは2本スラッシュで囲んで、最後にオプション（今回は「i」）を付けて表します。
今回の場合は、show_link内のテキストが/show/i（「show」や「Show」,「SHOW」など大文字小文字の違いを無視して判定）
と一致するかどうかを判定します。

**52行目**:expect(page).to have_button 'Create Book'
have_buttonではボタンが存在するかを判定します。
今回の場合はボタン内の文字列の'Create Book'を指定しています。

**57行目**:fill_in 'book[title]', with: Faker::Lorem.characters(number:5)
fill_inではフォームの値を変更します。通常はフォームにテキストを入力すると考えて問題ありません。
今回の場合name属性がbook[title]であるフォームを対象に指定しています。
with:のあとには入力したい文字列を指定します。
今回withの後にはFaker::Lorem.characters(number:5)としていて、5桁のランダムな文字列を入力する形になっています。
※Fakerはテスト用のダミーデータを作成してくれるもので、GemfileにてFakerを導入しています。

**59行目**:click_button 'Create Book'
click_buttonはボタンをクリックします。
ボタン内の文字列を指定します。

**60行目**:expect(page).to have_content 'successfully'
have_contentでは指定した文字列が含まれているかを判定します。
今回の場合はページ内に「successfully」という文字列があるかを判定します。

**71行目**:expect(page).to have_current_path book_path(Book.last)
have_current_pathでは現在のURLパスを取得します。
今回の場合は、投稿後のページURLが正しいURLパスであるかを判定しています。

**76行目**:expect{ book.destroy }.to change{ Book.count }.by(-1)
expectではbook.destroyとあり、データの削除をしています。波括弧になっていることに注意しましょう。
changeの値にBook.count（Bookのデータ数）を指定しており、by(-1)でそれが一つ減っているかという判定をしています。
このように記述することで、今どうなっているかだけではなく、どう変更されたかもチェックすることができます。

changeを使った例にはfrom,toによる記述の仕方もあります。
　~~~ruby
 x = 10
expect{x += 5}.to change{x}.from(10).to(15)
~~~
