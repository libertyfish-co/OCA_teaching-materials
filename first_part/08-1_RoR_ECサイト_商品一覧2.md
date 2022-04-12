## 8.1 Ruby on Rails：ECサイトの開発 商品一覧2

### 8.1.1 画面遷移とルーティングの実装

#### (b) 例題

以下は、地域(Area)テーブルをscaffoldしたあとのファイルです。まず、Controllerの `redirect_to` の部分で、createとeditのあとは直接一覧ページへ遷移するように変更しましょう。

`app/controllers/areas_controller.rb`

    # GET /areas/1 # 削除
    # GET /areas/1.json # 削除
    def show # 削除
    end # 削除

    def create
      @area = Area.new(area_params)

      respond_to do |format|
        if @area.save
          format.html { redirect_to areas_url,
                        notice: 'Area was successfully created.' } # 編集
          format.json { render :show, status: :created, location: @area }
        else
          format.html { render :new }
          format.json { render json: @area.errors,
                        status: :unprocessable_entity }
        end
      end
    end

    def update
      respond_to do |format|
        if @area.update(area_params)
          format.html { redirect_to areas_url,
                        notice: 'Area was successfully updated.' } # 編集
          format.json { render :show, status: :ok, location: @area }
        else
          format.html { render :edit }
          format.json { render json: @area.errors,
                        status: :unprocessable_entity }
        end
      end
    end

また、不要になった詳細ページに関連するView、ルート、テストは全て削除します。

`app/views/areas/show.html.erb`

    ファイルを削除

`config/routes.rb`

    resources :areas, except: :show # 編集

`test/controllers/areas_controller_test.rb`

    test "should show area" do # 削除
      get area_url(@area) # 削除
      assert_response :success # 削除
    end # 削除

#### (c) 問題

商品タグ(Tag)の登録と編集のあと、直接一覧ページへ遷移するように変更しましょう。また、不要になった詳細ページに関連するものは全て削除しましょう。
