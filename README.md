# useQuery + Axios Adapter

Esse padrão visa tornar as telas menos dependentes do Axios para realizar requisições à API. Com um adaptador, seguimos o Princípio de Substituição de Liskov do SOLID, permitindo trocar o Axios por outra biblioteca ou até mesmo pelo método fetch do JavaScript sem afetar o funcionamento das telas, o que simplifica futuras mudanças.

Centralizar todas as chamadas de API na pasta de serviços evita a duplicação de código. Ao criar funções para fazer essas chamadas e definir interfaces para elas, garantimos que não haja repetição em várias partes do código. Isso simplifica a manutenção e melhora a organização do projeto.

### Estrutura de pastas
```
src/
│
├── infra/
│   └── adapters/
│       ├── axiosAdapter.ts  
│       └── types.d.ts     
│
├── services/
│   ├── user/
│   │   ├── index.ts        
│   │   └── types.d.ts     
│   │
│   ├── api.ts
│   └── index.ts            
│
└── screens/
    └── User/
        └── List/
            ├── index.tsx   
            └── styles.tsx  
```

### Adapter

```js
import { AxiosError, AxiosResponse } from "axios";
import { catchHandler, thenHandler } from "../../utils/functions";
import { api } from "../../services/api";

export const axiosAdapter = async <T>({
  method,
  url,
  data,
  query,
  showSuccessToast = false,
}: IAxiosAdapter) => {
  let axiosResponse: AxiosResponse<T> | undefined;

  try {
    axiosResponse = await api.request({
      url: query ? `${url}?${new URLSearchParams(query)}` : url,
      method,
      data,
    });

    if (showSuccessToast)
      thenHandler(axiosResponse as AxiosResponse<{ message: string }>);
  } catch (error) {
    axiosResponse = undefined;
    const err = error as AxiosError<{ message: string }>;
    catchHandler(err);
  }

  return { response: axiosResponse?.data };
}
```

```js
type IAxiosAdapter = {
  url: string;
  method: "get" | "post" | "put" | "delete";
  data?: any;
  query?: {
    [key: string]: any;
  };
  showSuccessToast?: boolean;
};
```


### useQuery

```ts
  const { data, refetch, isLoading } = useQuery({
    queryKey: ["usersList", { offset, page, search: debouncedSearch }],
    queryFn: requestUsersList,
    staleTime: 1000 * 60, // Cache time: 60s
  });
```

### useMutation

```js
  const { mutateAsync, isPending } = useMutation({
    mutationFn: (userId: string) => requestDeleteUser({ userId }),
    onSuccess: () => refetch(),
  });
```
