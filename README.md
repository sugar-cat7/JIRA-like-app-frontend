JIRA Frontend

### 環境準備

- プロジェクトの作成 (typescript redux toolkit 使用)
  `npx create-react-app . --template redux-typescript`

- Package install

```
axios -> Rest apiと通信用
react-router-dom @types/react-router-dom -> ページ遷移用(typescript ver)
@material-ui/core @material-ui/icons @material-ui/lab -> style
```

- 環境変数定義
  - React のプロジェクトの場合.env 配下の環境変数は`REACT_KYE`みたいな感じで REACT をつける必要がある

### 作成するコンポーネントごとの Slice を作る

- What’s Slice

  - redux の action と reducer をまとめたもの
  - もともと一つの store で管理していたものを個々の slice という単位に分けて state を管理している感じ
    - 巨大な switch 文とかが無くなってだいぶ可読性が上がってる

- Slice で使うデータ型はまとめて features/types.ts に定義しておく(ts のみ）

```typescript
// authSlice.ts
export interface LOGIN_USER {
  id: number;
  username: string;
}

export interface FILE extends Blob {
  readonly lastModified: number;
  readonly name: string;
}

// ……
```

- slice の雛形

```typescript
import { createAsyncThunk, createSlice, PayloadAction } from "@reduxjs/toolkit";
import { RootState, AppThunk } from "../../app/store";
import axios from "axios";
import {
  AUTH_STATE,
  CRED,
  LOGIN_USER,
  POST_PROFILE,
  PROFILE,
  JWT,
  USER,
} from "../types";

const initialState: AUTH_STATE = {
  isLoginView: true,
  loginUser: {
    id: 0,
    username: "",
  },
  profiles: [{ id: 0, user_profile: 0, img: null }],
};

export const authSlice = createSlice({
  //slice作る
  name: "auth",
  initialState,

  reducers: {},
});

export const {} = authSlice.actions;

//need to change
export const selectCount = (state: RootState) => state.counter.value;

export default authSlice.reducer;
```

- API を叩きに行く非同期関数例

```typescript
export const fetchAsyncLogin = createAsyncThunk(
  // createAsyncThunkとは非同期処理の実行状況に応じたActionCreatorを生成する関数
  // createAsyncThunkはあくまでActionCreatorを返すだけ、ActionCreatorに対応したReducerを別途実装する必要
  // -> extra reducer内
  "auth/login", //actionの名前　一意にする
  async (auth: CRED) => {
    const res = await axios.post<JWT>( //返り値はジェネリクスで型指定できる
      `${process.env.REACT_APP_API_URL}/authen/jwt/create`,
      auth,
      {
        headers: {
          "Content-Type": "application/json",
        },
      }
    );
    return res.data;
  }
);
```

- 非同期関数の後処理(extrareducer)

```typescript
extraReducers: (builder) => {
  //非同期関数の後処理用
  builder.addCase(
    fetchAsyncLogin.fulfilled, //thunkからpromise型で帰ってきているので状態を見ている
    (state, action: PayloadAction<JWT>) => {
      //非同期関数のreturn値がpayloadに入ってる
      localStorage.setItem("localJWT", action.payload.access); //アクセストークンをローカルストレージへ
      action.payload.access && (window.location.href = "/tasks"); //OKならぺーじ遷移
    }
  );
  //….
};
```

- store に追記

```typescript
import authReducer from "../features/auth/authSlice";
export const store = configureStore({
  reducer: {,
    auth: authReducer,
  },
});
```

State の情報を確認するには Chrome の Redux Devtool を使えば良い
￼

### コンポーネントを作る

```typescript
const Auth: React.FC = () => {
  const classes = useStyles();
  const dispatch: AppDispatch = useDispatch();
  const isLoginView = useSelector(selectIsLoginView);
  const [credential, setCredential] = useState({ username: "", password: "" });
  // textfieldに入力されるたびにsetcredentialが呼び出されstateが更新される
  const handleInputChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const value = e.target.value;
    const name = e.target.name;
    setCredential({ ...credential, [name]: value });
  };
  // selectorで呼び出しておいたログイン情報をもとに実行する関数を決定
  const login = async () => {
    if (isLoginView) {
      await dispatch(fetchAsyncLogin(credential));
    } else {
      const result = await dispatch(fetchAsyncRegister(credential));
      if (fetchAsyncRegister.fulfilled.match(result)) {
        await dispatch(fetchAsyncLogin(credential));
        await dispatch(fetchAsyncCreateProf());
      }
    }
  };

  return <></>;
};
```
